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
sudo apt-get install bluez bluez-utils bluez-tools libbluetooth-dev python-dev
```

Install btproxy:

```bash
git clone https://github.com/conorpp/btproxy
cd btproxy
sudo python setup.py install
```

# Running it on the Pebble Watch

Now to run it, you will need two Bluetooth devices to proxy. 
I choose to use my Phone (Nexus 6) and Pebble Steel watch.

So I went ahead and made each device Bluetooth discoverable.  For
the Nexus 6 running Android L, this just means opening Bluetooth
in the settings.  For the Pebble watch, you also just open Bluetooth
in the settings.

Now that they are visible, the Proxy can run.

First I use `hcitool` to scan for the devices so I know their Bluetooth MAC addresses.

```bash
$ hcitool scan
```
```text
Scanning ...
        77:88:99:AA:BB:CC   Pebble 9FAA
        11:22:33:44:55:66   conorpp's Nexus 6
```

Now to run the Bluetooth proxy.

```bash
sudo btproxy 11:22:33:44:55:66 77:88:99:AA:BB:CC
```

Notice I put the phone's MAC address first.  This is important because the
phone is the master device in the connection and the watch is the slave.
This master/slave setup is part of the Bluetooth protocol.  Slave devices
are typically the ones to sit and advertise until a master device requests
for it's connection.  A master can connect to multiple devices while a slave
can only have one connection.

![](/assets/images/btproxy/master_slave.png "These are called piconets")
*credit: sparkfun.com*

Now let's follow the output of the proxy.

```bash
$ sudo btproxy 11:22:33:44:55:66 77:88:99:AA:BB:CC
```
```
Running proxy on master  11:22:33:44:55:66  and slave  77:88:99:AA:BB:CC
Using shared adapter
Slave adapter:  hci0
Master adapter:  hci0
Looking up info on slave (77:88:99:AA:BB:CC)
Looking up info on master (11:22:33:44:55:66)
Spoofing master name as  Pebble 9FAA_btproxy
Running inquiry scan
paired
Spoofing master name as  Pebble 9FAA_btproxy
Proxy listening for connections for "Serial Port Server Port 1"
Proxy listening for connections for "Audio/Video Remote Control"
Attempting connections with 2 services on slave
Connected to service "Audio/Video Remote Control"
Connected to service "Serial Port Server Port 1"
Now you're free to connect to "Pebble 9FAA_btproxy" from master device.
```

The Proxy will look up the name and class of the devices about to be proxied
so it can copy the name and class to the Bluetooth adapters being used.  In this
case only one adapter is being used so it will copy the slave's properties.

Halfway through this it made a pairing request to my Pebble watch, to which I accepted.
The proxy then opens Bluetooth sockets for each service that the watch hosts, which will be
what the phone connects to.  The proxy connects to the services on the watch.  Once this is done,
I connect the master device (my phone) to the proxy device (Pebble 9FAA_btproxy).

![Connect on Phone and Pebble app](/assets/images/btproxy/nexus_connect.jpg)
![Connect on Phone and Pebble app](/assets/images/btproxy/pebble_connect.jpg)

```
Accepted connection from  ('11:22:33:44:55:66', 1)
>>  b'\x00\x1e\x001\x01\x1b\x07\xe0\xd9\xcb\x89WK\xf7\x9dB5\xbfG\xca\xad\xfe\x01\x01\x00\x00\x00\x02\x04\x00\x01\x00\x00\x00\x00\x01\x00\x11\x00'
<<  b'\x00\x11\x00\x11\x01\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00\x00\x02\x02\x02\x04\x00'
<<  b'\x00\x01\x00\x10\x00'
>>  b'\x00\x96\x00\x10\x01U\t\xb4\xfbv2.9.1\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x0054664bd\x00\x00\x06\x01R"T_v1.5.5\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x001c16275\x00\x01\x06\x01R\xe2\xf82102V1\x00\x00\x00\x00Q206134E01L3\xaa\x9f\xa6\xe9\x17\x00&e\x8a\x03U\t\xb4\xfben_US\x00\x00\x01XXXXXXX\x00'
<<  b'\x00\x0b\x13\x89\x00\tmfg_color'
>>  b'\x00\x06\x13\x89\x01\x04\x00\x00\x00\x07'
<<  b'\x00\x01\x17p\x01'
>>  b'\x00\t\x17p\x01\x00\x00\x00\x08\x00\x00\x00\x00'
<<  b'\x00\x05\x00\x0b\x02U\xec^4'
```

The protocols getting dumped here are SPP and Pebble's own protocol.  They are binary protocols which is why it's unreadable.
But sending a notification should come up as clear text.  Here is what gets logged after sending myself a text message:

```
<<  b'\x00F\x0b\xc2\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00K_\xecU\x01\x02\x03\x01\x0e\x00(123) 456-6789\x03\x0c\x00Hello Pebble\xff\x05\x00\xfe\x05\x00\x01\x03\x01\x01\x05\x00Reply'
```

You can see the phone number and the contents of the text message, "Hello Pebble".

Here's the packet sent after I sent myself a Google Hangouts message:

```
<<  b'\x00b\x0b\xc2\x00\x01\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\xf2_\xecU\x01\x02\x02\x01\r\x00Conor Patrick\x03\x1a\x00Sent a message on Hangouts\x01\x02\x01\x01\r\x00Open on phone\x02\x04\x01\x01\x07\x00Dismiss'
```

You can see it sent two options: "Open on phone" and "Dismiss".

Anything in the packets can be altered in real time using an inline Python script.  For example, here is
and inline script that will change the notification options in the Pebble's packets.

```python
def master_cb(req):

    req = req.replace(b'Open on phone', b'Hi welcome to')
    req = req.replace(b'Dismiss', b'Btproxy')

    print( '<< ', repr(req))
    return req

def slave_cb(res):
    print('>> ', repr(res))
    return res
```

`master_cb` is called when the master device (phone) is sending a packet to the slave (watch).  And vice versa with `slave_cb`.

Resulting display on Pebble:

![Active packet manipulation](/assets/images/btproxy/pebble.jpg "Active packet manipulation")










{% include JB/mytwitter %}
