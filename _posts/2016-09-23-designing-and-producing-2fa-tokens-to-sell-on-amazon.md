---
layout: post
title: "Designing and Producing 2FA tokens to Sell on Amazon"
description: ""
category:
tags: []
image: /assets/images/default.jpg
---
{% include JB/setup %}

I made a two factor authentication token and have made it [available on Amazon](https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6/).
In this post I'll talk about the design, how I affordably produced it, and some
metrics about selling on [Amazon](https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6/).  If you're interested in doing something similar,
you can copy everything as it's all open source.

![](/assets/images/u2f/crab_bowl.jpg)

# The Design

It uses the U2F protocol, which is a standard developed by the FIDO Alliance and Google.
U2F uses challenge response for authentication and is based on the P256 NIST Elliptic Curve.
FIDO additionally provides U2F standards for transports like USB, Bluetooth, and NFC which
makes a project like this ideal.

I choose to use the following components to implement the design (in order of importance):

* [ATECC508A](http://www.digikey.com/product-detail/en/atmel/ATECC508A-SSHDA-B/ATECC508A-SSHDA-B-ND/5213053) - Atmel I2C peripheral that implements P256 signatures and full key life cycle securely.
* [EFM8UB1](http://www.digikey.com/product-detail/en/silicon-labs/EFM8UB11F16G-C-QSOP24/336-3411-5-ND/5592439)   - Cheapest microcontroller with a USB slave peripheral and enough memory to implement U2F.
* [RGB LED](http://www.digikey.com/product-detail/en/LTST-C19HE1WT/160-2162-1-ND/4866310)   - Give user indication of state.
* Other discrete components - Button, bypass capacitors, ESD protection, current limiting resistor.

![](/assets/images/u2f/u2fzero.jpg)

I decided that for a U2F token to be secure it would need to meet 3 requirements:

1. Needs a good source of randomness to generate keys.
2. Strong computation for the crypto.  Only using an 8 bit processor would be too time consuming.
3. Tamper resistance.  I.e. it should be hard to duplicate.

The ATECC508A chip fulfills all these tasks becasuse it has a hardware RNG, write only keys, and hardware
acceleration for elliptic curve operations.

You can see the full source of the design [here](https://github.com/conorpp/u2f-zero).

One note on finding a secure chip: 

Finding a chip that has secure public key crypto (P256) implementations is non-trivial.  I got
lucky when I found Atmel's chip which was easy to purchase and get documentation for.  Other manufacturers
that offered potential secure chips were NXP, ST Electronics, and Infineon.  But none of them sell their secure
chipsets on normal distributers and seem to require customers to go through NDAs, licensing, and large minimum order
requirements.  Starting out, all of that is over my head.  I hope the market for public key hardware becomes more transparent.


# Producing it

This is the fun part.  I was originally hopeful when starting this project that I could be able to
solder everything in one night.  But that proved to be too time consuming, messy, and unreliable.  So
I got it fab'd and assembled at [PCBCart](http://www.pcbcart.com/).  But the real challenge here was programming
the tokens.

Programming was a challenge because it had to be integrated with my own scripts to handle the initial configuration
of the ATECC508A chip and generation and signing of an attestation certification per the U2F spec.  In other words,
one binary blob firmware did not cut it.  Each token needed to be programmed once to be "customized" and then programmed
again with a signed* build.  It took me about 1-2 minutes to program a U2F token while prototyping.

Manufacturers like PCBCart typically offer mass programming services but my process was too complicated and would
have ended being untrustworthy and expensive.

So I decided to program everything on my own while promising to my future self that I would make a pipelined process to
get everything done cheaply in a reasonable amount of time.

Fast forward a couple months, I have tons of U2F tokens from PCBCart:

![](/assets/images/u2f/box.jpg)

I automated and optimized my programming setup.  I acquired 3 programmers, 3 USB extension cables, and made 3
connecters using protoboard and machine pins.

![](/assets/images/u2f/mounted.jpg)

How it would work:

1. I plug in a U2F token to a USB cable for power and plug in a programmer.
2. I press a button on my keyboard to start programming on that programmer.
3. The script programs a setup file with a unique serial number so it can
be communicated with in parallel with other tokens.
4. The configuration and signing occurs.
5. A final program is built and programmed.
6. The token is told to blink green and blue so I can see that it is done.

![](/assets/images/u2f/leds.jpg)

This process would take about 10 seconds.  Because I had three set ups, I could
get up to three working at the same time.  It would have been quicker with 4 or 5, but
eventually I wouldn't be able to plug them in fast enough to get more then 5 programming
at the same time.

![](/assets/images/u2f/programming_long.gif)

It took me about 4-6 hours total to get through everything.  I watched two
movies (props if you can figure out which ones from the gif).  Occasionally my
pipeline would stall on some edge cases and I would have to restart but it was
mostly smooth.  I only had 1 token from PCBCart that didn't work (which is okay
since PCBCart had a couple extra).


* The public key in the build is signed for attestation.

# Fulfillment by Amazon

My goal was to have this listed in stock for at least a year.  I really don't
know what to expect on selling so it's a bit of a shot in the dark.  I know I don't
want to be bothered handling payments and shipping so going with a distributer is
needed.  I decided to go with Amazon because they make it easiest for most people
to order things online (and free shipping).  I'm not sure about comparing fees with other
options like [Tindie](https://www.tindie.com/).

Typically to list a product on Amazon, it is very easy if it's already listed.  You just
pick the already listed product and reuse all of the information already there.

But if you're adding a new product, you have to consider a couple options:

1. New Brand Owner:  If you own the brand of a product not currently on Amazon,
you can apply for Amazon to approve your brand and then list the product.
2. New product but not brand owner:  If you want to list a product you do *not* own
the brand for, you need to get permission from the brand owner to sell it and then Amazon
will allow you to list the product.

Because I'm the owner of the brand (I called it U2F Zero), I of course attempted the first option.






{% include JB/mytwitter %}
