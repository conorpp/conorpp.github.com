---
layout: post
title: "Linx: the filesharing server every hacker should know about"
description: ""
category: 
tags: [Tools]
image: /assets/images/linx/cover.png
---
{% include JB/setup %}

# Linx

Many file sharing servers already exist; a lot of them come and go.  Services Google Drive and Drop Box
  are the big ones.  Also there's have more lightweight options like pastebin or 
  `python -m SimpleHTTPServer`.

[Linx](https://github.com/andreimarcu/linx-server) is a recent, open source file sharing server 
that I've recently started using.  It makes it easy for sharing files or quickly written scripts 
that won't be tied to any public account. It's also nice if I'm hacking on a project and don't want 
to deal with setting up large web scale file storing accounts/API's (AWS, Firebase, etc).  

So Linx really sticks out for the following reasons:

* It's written in Go (which seems to be a plus these days).
* Has a built in [API](https://linx.li/API/) that is as simple as a file sharing API *should* be.
* It supports displaying common file types with proper syntax highlighting if needed:
  * jpg, pdf, txt, c, python, shell, contents of compressed files, etc.
* No user accounts or bullshit.  Basic anonymity is attained with randomized filenames and file expirations.
* It's [open source](https://github.com/andreimarcu/linx-server) and was designed for other people to run.

# Setting up your own Linx server

Right now I'm running [my own Linx server](https://0x123.xyz/) on one of my Digital Ocean instances.

![](/assets/images/linx/term.png)

It was pretty easy to set up since there are [builds for all majors platforms](https://github.com/andreimarcu/linx-server/releases) (except mobile).  It has all the configuration details I care about and nothing more.  For my setup, I have it sitting behind my Nginx server with fast-cgi and HTTPS.

Easiest way to setup (64-bit Linux):

```bash
wget https://github.com/andreimarcu/linx-server/releases/download/v1.1.4/linx-server-v1.1.4_linux-amd64
./linx-server-v1.1.4_linux-amd64
```

You should pick out the [build for your system](https://github.com/andreimarcu/linx-server/releases) and 
try it out.
If you're setting up your own Linx server for real, make sure you cover a couple configurations:

```bash
./linx-server -bind 0.0.0.0:8080 \
  -siteurl "http://0x123.xyz/" \
  -remoteuploads \
  -maxsize 1048576
```

If you want Linx to be public, you should tell it to bind to `0.0.0.0`.  It's
also very important to let Linx know the `siteurl` - not only so Linx can
correctly generate links, but also to prevent hotlinking by checking the origin or referer.  

Hotlinking is sometimes a serious
problem that small services face.  As we can see from The Oatmeal,
it can seriously increase server bills.

![seriously increase server bills](/assets/images/linx/hot.jpg)

Wouldn't have been a problem if the image was hosted with Linx!


## More features of Linx:

* Built in https/TLS server
* API key authentication and generation
* Content security policies
* HTTP proxy support
* Torrenting support

# Demo

Here is a shell script for geolocating IP addresses I wrote some time ago:

* https://0x123.xyz/geoip.sh

A barcode from a to-go food container at my school [PDF]:

* https://0x123.xyz/to-go-container-barcode.pdf

A picture of a deer I took once:

* https://0x123.xyz/3naturedeer.jpg

You're free to [upload](https://0x123.xyz/) anything to my server under that is under 5 MiB.

# Conclusion

Linx has everything I want from a file sharing server.
If there's something that's missing, you should [contribute](https://github.com/andreimarcu/linx-server#development).




{% include JB/mytwitter %}
