---
layout: post
title: "Proxying Bluetooth devices for security analysis using btproxy"
description: ""
category: Tools
tags: []
image: /assets/images/default.jpg
---
{% include JB/setup %}

# btproxy

I've recently been interested in investigating the security of new technologies.  New devices,
especially in the "IoT" realm, use Bluetooth to communicate.  I got interested finding a good
way to get insight to Bluetooth connections.  I wanted to be able to see traffic in clear text
and modify it actively, like many tools allow you to do with internet traffic.  However, there
isn't really any cheap and easy methods for doing so.

I wrote a tool that will leverage 1 or 2 Bluetooth adapters to act as a proxy for two other
devices connecting to each other.  Proxying the connection allows insight into clear text
traffic and the ability to modify it in real time.

# Installation

The code currently lives on Github and only works on Linux or OS X.  It relies on BlueZ.

Install the dependencies:

```bash

```

Install btproxy:

```bash
git clone https://github.com/conorpp/btproxy
cd btproxy
```



{% include JB/mytwitter %}
