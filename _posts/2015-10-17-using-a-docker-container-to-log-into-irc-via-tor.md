---
layout: post
title: "Using a Docker container to log into IRC via Tor"
description: ""
tags: [Tools]
image: /assets/images/default.jpg
---
{% include JB/setup %}

TL;DR
=====
For those of you that use Docker, [or would like to learn](https://docs.docker.com/linux/started/),
you can immediately get started by running:

```bash
docker run -it cpp1/weechat_tor
```

It will pull a container containing Ubuntu 14.04, [WeeChat](https://weechat.org/) (IRC client), and a Tor proxy.
It opens up WeeChat and immediately connects to OFTC over Tor.

You'll see something like this:
![](/assets/images/irc/weechat_connect.png "IRC over Tor with Weechat")

Docker
======
**Nutshell**: Docker is an open-source project that leverages isolation features in the Linux kernel to run
applications in VM-like userspace environments (containers).  A container is often a more lightweight and
efficient option than running an application in a virtual machine.

I've recently got hooked on it because it's great for reusing applications that have some complexity
to the environment they run in.  For example, using Weechat + Tor is a nice service I like to have running
for myself, but it's a bit of a pain to set up each time I move to a new computer or server.

The [Virginia Tech Cyber Security Club](http://vtcsec.org/) is planning on hosting a collegiate 
[CTF event](http://conorpp.com/blog/how-to-fix-a-corrupted-file-by-brute-force/#the-challenge) next year
and I'm vouching for challenges to be made in Docker.  That way any student can make a challenge, and by
putting it in a Docker container, makes it much easier to host and setup for the event.

WeeChat
=======
[WeeChat](https://weechat.org/) is a feature rich IRC client that's nice because it uses ncurses and can easily be used in a terminal.
I just leave it running on a server.  It's nice to leave your IRC client running continuously somewhere so you can drop in on your channels
any time without having to reconnect or lose back logs.

Also seeing a lot of these can get annoying:
![](/assets/images/irc/disconnect.png "Damn it get your set up right")

I personally like to use Digital Ocean and tmux.  *Really* easy to set up.

```bash
# server:
tmux new -s irc
docker run -it cpp1/weechat_tor
# close terminal

# from laptop/desktop:
ssh username@myserver.xyz -t tmux a -t irc
```
P.S. [Students can get $50 credit on Digital Ocean though Github](https://education.github.com/pack).

Tor
===
[Tor](https://www.torproject.org/) is a very popular and awesome anonymity network.
But why need to be anonymous?  I don't have much reason to other then to show off for this post.  However,
it was great for [checking out a botnet](http://conorpp.com/blog/a-close-look-at-an-operating-botnet/).

Resources
===============
You can check out my Dockerfile [here](https://github.com/conorpp/Dockerfiles/tree/master/Weechat_Tor) if you want to make changes.




{% include JB/mytwitter %}
