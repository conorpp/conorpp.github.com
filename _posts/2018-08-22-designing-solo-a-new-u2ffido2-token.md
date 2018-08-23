---
layout: post
title: "Designing Solo, a new U2F/FIDO2 Token"
description: ""
category: 
tags: [Startup]
image: /assets/images/solo/cover.jpg
---
{% include JB/setup %}


For the past couple years, [I’ve been selling U2F tokens on
Amazon](https://conorpp.com/designing-and-producing-2fa-tokens-to-sell-on-amazon).
I’ve been selling an average of 150 units per month and have ordered multiple
batches over the past two years.  

Due to recent developments with FIDO2, WebAuthn, and various features
people have been requesting, I’ve started an upgrade for U2F Zero.  A couple
collaborators and I have decided to call it Solo (or Solo Key).  It is the sole
thing you need to secure your accounts :).

We also plan to [bootstrap this by getting funding on Kickstarter](https://solokeys.com/).  If you’re
interested, sign up below!

<div id="mc_embed_signup" style="display: inline-block">
<form action="https://solokeys.us19.list-manage.com/subscribe/post?u=cc0c298fb99cd136bdec8294b&amp;id=b9cb3de62d" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate form-inline" target="_blank" novalidate="" _lpchecked="1">
<div class="mc-field-group">
<label for="mce-EMAIL" style="display: none">Email address</label>
<input type="email" value="" name="EMAIL" class="required email form-control" id="mce-EMAIL" placeholder="Email address">
</div>
<div id="mce-responses" class="clear">
<div class="response" id="mce-error-response" style="display:none"></div>
<div class="response" id="mce-success-response" style="display:none"></div>
</div>    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
<div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_cc0c298fb99cd136bdec8294b_b9cb3de62d" tabindex="-1" value=""></div>
<div class="clear"><input type="submit" value="Join waitlist" name="subscribe" id="mc-embedded-subscribe" class="button btn btn-primary"></div>
</form>
</div>

# Solo


The original U2F Zero was just a plain circuit board that only did U2F over USB.  Solo will have the following features.

- USB-A and USB-C option
- NFC support for mobile devices
- Flexible + durable case that embeds a tactile button
- Various color options
- FIDO2 protocol support
- Easy, web-based secure update process
- Good documentation for everyday users

FIDO2 is exciting since there will be a lot more adoption for it and it's planned to be a password replacement.  For example, you can use a FIDO2
authenticator to log on to Windows, out of the box.

![](/assets/images/solo/colors.jpg "Colors!")

And of course, everything will soon be open sourced like with U2F Zero.

# Design

Development started completely in software, no hardware, to make sure it could
be easily contributed to by others.  Most coding and testing can be done solely
on a PC, while being designed to be easily ported to other chips, like the
NRF52840 by Nordic Semiconductor and EFM32J by Silicon Labs.

PC-only testing is achieved by patching Yubico FIDO2 Python API to exchange
messages with a PC application instead of a USB HID device.

After verifying a "first draft" of the FIDO2 implementation, I ported the design
to an NRF52840 development board and to a Silicon Labs EFM32J development
board.

![](/assets/images/solo/nrf.jpg "NRF52840"){:height="45%" width="45%"}
![](/assets/images/solo/efm32.jpg "EFM32J"){:height="45%" width="45%"}

NRF52840 Bluetooth, NFC, USB SoC		EFM32J + EFM8UB1

I ultimately decided to go with the EFM32J.  It consumes less power, is much
cheaper, and is easier to solder (QFN32 vs BGA).  NRF52840 initially seemed
like a great solution since it has everything in 1 chip, but it brings more
complication and cost.  Plus it would make it much harder for people to solder
their own Solo key.

The design is modular in that USB and NFC support is added by adding another
chip.  The EFM8UB1 provides the USB HID interface, and the AMS 3955/3956
provides the passive NFC interface.  NFC also requires a coil, which will be an
external coil that lies flat next to circuit, all in case.  More on this later.


## Hardware key isolation


Providing some sort of hardware isolation for secret key material is hard since
most of the chips on the market that can do that, require a NDA and that the
company be a bank, government, or other reputable entity.

There are the ATECC508A and ATECC608A crypto chips, which can be obtained
easily.  U2F Zero actually uses the ATECC508A for key isolation and crypto
acceleration.  But since U2F/FIDO2 devices need to derive secret keys at
runtime to be able to work with an unlimited number of services, the ATECC508A
doesn’t work out well.  This is because to derive a key at runtime, you
typically need to compute an HMAC that’s keyed with a master secret, and store
the result as the runtime key.  This needs multiple interactions with the
ATECC508A and requires the runtime key be stored temporarily on the MCU, which
tarnishes the idea of key isolation.

I’m not aware of any other attainable crypto chips that can do better than
this.

It’s best to keep the design simple rather than lob in a security chip that
isn’t a good fit.  The main threat is adverse software on the computer you plug
your token into.  Solo’s firmware isn’t complex, so it is reasonable to try to
avoid buffer overflows and similar exploits.  If a critical bug is found, a
signed update process is supported so it can easily be patched.  This model
generally works well in industry (Trezor, the popular bitcoin wallet, is
successful with this approach) and greatly simplifies the design of Solo.

## Hardware design


Previously I’ve only used Kicad for PCB layout, to include the design of U2F
Zero.  For Solo, I basically did a token design in Kicad, Eagle, and Circuit
maker to figure out what is best.  I stuck with Eagle in the end, because of
the nice integration with Fusion360, which really helps the mechanical design.

![](/assets/images/solo/layout.png "Eagle"){:height="50%" width="50%"}

Layout done in Eagle.

![](/assets/images/solo/pcbrender.png "This is great"){:height="50%" width="50%"}

3D model in Fusion 360.

![](/assets/images/solo/pcb_jig.png "Quick to design a jig"){:height="50%" width="50%"}

Custom programming jig design.


![](/assets/images/solo/jig.jpg){:height="30%" width="30%"}
![](/assets/images/solo/pcbinjig.jpg){:height="30%" width="30%"}
![](/assets/images/solo/ledon.jpg){:height="30%" width="30%"}

Working first revision of 3D printed jig and prototyped PCB.

## The case


We want our device to be something people like to use and carry around (unlike
most 2FA options out there).  So getting the look and feel of it right is
important.

We have two designs for the case.  One can be printed on a SLA printer ($2.50
from DirtyPCBs), and the other we are getting professionally molded.  Both
designs take advantage of a thin mechanical button that provides nice tactile
feedback.  

The additively manufactured design allows people to produce their own.  It
consists of a top and bottom piece that snap together over the circuit board.
The bottom piece also removes the 2mm requirement for PCBs.

![](/assets/images/solo/case2render.png){:height="45%" width="45%"}
![](/assets/images/solo/case2.jpg "3D Printed"){:height="45%" width="45%"}
![](/assets/images/solo/2piece.jpg){:height="50%" width="50%"}

The molded design will be a semi-hard silicone “sock”.  It will be
pocket-friendly, nice to hold, and allow embedded mechanical button to be used.
Multiple colors can be offered.

![](/assets/images/solo/case1render.png){:height="45%" width="45%"}
![](/assets/images/solo/case1.jpg "Also 3D printed, but will be molded."){:height="45%" width="45%"}

The design is still being iterated on and current photos are from a 3D printer.

## USB-C


Two versions of the PCB will be produced to support USB-A and USB-C connectors.

![](/assets/images/solo/usbc.png){:height="50%" width="50%"}

We will aim to re-use the same case for USB-A and USB-C.

## NFC Support

For a long time, I thought passive NFC support wasn’t going to work out.  The
only MCUs out there that can run passively on an NFC interface, and communicate
bidirectionally, are aforementioned forbidden security chips.  But there are a
couple of chips out there that just provide an NFC interface, and an energy
harvesting capability.

The AMS 3956 is one good example.  AMS 3956 is extra special because it allows
for NFC type 4 emulation, a requirement for U2F/FIDO2.  Other energy harvesting
ICs, like the NXP NT3H2111, just implement NFC type 2.

It is also critical to use a low-power MCU to pair with the AMS 3956, like the
silabs EFM32J, because the harvested NFC power is pretty low, around 3-5 mA at
3V in my tests.

It will still take more design effort to finish the NFC version.  Some low
level NFC communication details need to be implemented, and the coil needs to
be designed and tuned.

## Signed firmware updates through browser


Signed firmware updates are supported via an extension to the U2F/FIDO2
protocol.  No extra software needs to be installed.  In case any critical bugs
are discovered, it will be very easy to update.

Users will be able to get any new features added by me or the community.
Developers will be able to reprogram the firmware using a hardware debugger.

![](/assets/images/solo/update.png "Updating is a matter of holding down the button and visiting a website")

# Kickstarter


We need some funding to be able to bootstrap this.  To make this project
affordable, i.e. reaching scale on both the PCBs and molded case designs, we
would like to be able to order 10,000 units.  We could get away with like 3000,
but our numbers really start to look sustainable at 10,000.

A successful crowdfunding campaign would really help get this project on the
market.  We would need a minimum of 1000 backers, but we really start to shine
if we can get around 4000.  Assuming an average 2.5 “base” units per backer, we
could reach our 10,000 units sold goal.

From there, we would be in a great place to continue working on making a great
2FA/1FA device for the masses.

![](/assets/images/solo/twitter.jpg "If you like this, back our Kickstarter!")


Interested?  Sign up to our mailing list, we will only email you when our
Kickstarter campaign starts.

<div id="mc_embed_signup" style="display: inline-block">
<form action="https://solokeys.us19.list-manage.com/subscribe/post?u=cc0c298fb99cd136bdec8294b&amp;id=b9cb3de62d" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate form-inline" target="_blank" novalidate="" _lpchecked="1">
<div class="mc-field-group">
<label for="mce-EMAIL" style="display: none">Email address</label>
<input type="email" value="" name="EMAIL" class="required email form-control" id="mce-EMAIL" placeholder="Email address">
</div>
<div id="mce-responses" class="clear">
<div class="response" id="mce-error-response" style="display:none"></div>
<div class="response" id="mce-success-response" style="display:none"></div>
</div>    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
<div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_cc0c298fb99cd136bdec8294b_b9cb3de62d" tabindex="-1" value=""></div>
<div class="clear"><input type="submit" value="Join waitlist" name="subscribe" id="mc-embedded-subscribe" class="button btn btn-primary"></div>
</form>
</div>

<!--Suggestions / comments / questions ?  Reach out below, on Twitter, or by email!-->


{% include JB/mytwitter %}
