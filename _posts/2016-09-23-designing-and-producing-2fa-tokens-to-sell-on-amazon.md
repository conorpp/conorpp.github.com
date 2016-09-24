---
layout: post
title: "Designing and Producing 2FA tokens to Sell on Amazon"
description: ""
category:
tags: [Startup]
image: /assets/images/u2f/u2fzero.jpg
---
{% include JB/setup %}

I made a two factor authentication token and have made it [available on Amazon](https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6/).
In this post I'll talk about the design, how I produced it affordably, and some
metrics about selling on Amazon.  If you're interested in doing something similar,
you can copy everything as it's all open source.

![](/assets/images/u2f/crab_bowl.jpg)

# The design

It uses the [U2F protocol](https://fidoalliance.org/specifications/download/), which is a standard developed by the FIDO Alliance and Google.
U2F uses challenge response for authentication and is based on the P256 NIST Elliptic Curve.
FIDO additionally provides U2F standards for transports like USB, Bluetooth, and NFC which
makes a project like this ideal.

I decided that for a U2F token to be secure it would need to meet 3 requirements:

1. A good source of randomness to generate keys.
2. Strong computation for the crypto (using an 8 bit processor would be too time consuming).
3. Tamper resistance.  Hard to duplicate.


I chose to use the following components to implement the design (in order of importance):

* [ATECC508A](http://www.digikey.com/product-detail/en/atmel/ATECC508A-SSHDA-B/ATECC508A-SSHDA-B-ND/5213053) - Atmel chip that securely implements P256 signatures and key generation \*.
* [EFM8UB1](http://www.digikey.com/product-detail/en/silicon-labs/EFM8UB11F16G-C-QSOP24/336-3411-5-ND/5592439)   - Cheapest microcontroller with USB.
* [RGB LED](http://www.digikey.com/product-detail/en/LTST-C19HE1WT/160-2162-1-ND/4866310)   - Better user experience.
* Other discrete components - Button, bypass capacitors, ESD protection, current limiting resistor.

![](/assets/images/u2f/u2f_diagram.png)

The ATECC508A chip fulfills all security requirements because it has a hardware RNG, write only keys, and hardware
acceleration for elliptic curve operations \*\*.

You can see the full source of the design [here](https://github.com/conorpp/u2f-zero).  

It would be much better
with a tamper resistant case or coating, but the initial capital to get that going is currently out of my reach.

# Producing it

This is the fun part.  I was originally hopeful when starting this project that I would be able to
solder everything in one night.  But that proved to be too time consuming, messy, and unreliable.  So
I got it fab'd and assembled at [PCBCart](http://www.pcbcart.com/).  But the real challenge was programming
the tokens.

Programming was a challenge because it's dependent on my scripts to handle the initial configuration
of the ATECC508A chip and creation of a unique attestation certification per U2F token.  In other words,
one binary firmware blob did not cut it.  Each token needed to be programmed once to be "customized" and then programmed
again with a signed build \*\*\*.  It took me about 1-2 minutes to program a U2F token while prototyping.

Manufacturers like PCBCart typically offer mass programming services but my process was too complicated and would
have ended being untrustworthy and expensive.

I decided to program everything on my own while promising to my future self that I would make a pipelined process to
get everything done cheaply in a reasonable amount of time.

Fast forward a couple months, I have a lot of U2F tokens from PCBCart:

![](/assets/images/u2f/box.jpg)

I automated and optimized my programming setup.  I acquired 3 programmers, 3 USB extension cables, and made 3
connectors using protoboard and machine pins.

![](/assets/images/u2f/mounted.jpg)

How it would work:

1. I plug in a U2F token to a USB cable for power and plug in a programmer.
2. I press a button on my keyboard to start programming on that programmer.
3. The script programs a setup firmware with a unique serial number so the token can
be communicated with in parallel with other tokens.
4. The configuration and signing occurs.
5. A final program is built and programmed.
6. The token is told to blink green and blue so I can see that it is done.

![](/assets/images/u2f/leds.jpg)

This process takes about 10 seconds per token.  Because I had three setups, I could
get up to three working at the same time.  It would have been quicker with 4 or 5, but
eventually I wouldn't be able to plug them in fast enough to get more then 5 programming
at the same time.

![](/assets/images/u2f/programming_long.gif)

It took me about 4 hours total to get through everything.  I watched two
movies (props if you can figure out which ones).  Occasionally my
pipeline would stall on some edge cases and I would have to restart but it was
mostly smooth.  I only had 1 token from PCBCart that didn't work (which is okay
since PCBCart made a couple extras).



# Fulfillment by Amazon

My goal was to have U2F Zero listed in stock for at least a year.  I don't know
how many I can expect to sell so it's a bit of a shot in the dark.  I know I don't
want to be bothered handling payments and shipping so going with a distributer is
needed.  I decided to go with Amazon because they make it so easy for most people
to order things online (and free shipping).  


Typically to list a product on Amazon, it's very easy if it's already listed.  You just
pick the already listed product and reuse all of the information already there.

But if you're adding a new product, you have to consider a couple options:

1. **New Brand Owner**:  If you own the brand of a product not currently on Amazon,
  you must apply for Amazon to approve your brand and then list the product.

2. **New product but not brand owner**:  If you want to list a product but you do *not* own
  the brand for it, you need to get permission from the brand owner to sell it and then Amazon
  will allow you to list the product.

Because I was the owner of the brand (I called it U2F Zero), I attempted option (1).
However, unmentioned in their documentation, Amazon requires that the brand be
printed directly on the packaging to protect the brand.  Stickers or labels do
not count.  All I had and had planned to use was polybags and labels.

![](/assets/images/u2f/polybag.jpg)

I didn't want to back up and get everything repackaged with my
"brand" printed on it.  I decided that the owner of the U2F Zero
"business" would be my LLC -- ConorCo.

I wrote a letter to ConorCo LLC and asked for permission to sell their
product, U2F Zero, on Amazon.  The majority shareholder of ConorCo LLC, Conor
Patrick, promptly signed off to give me permission to do that.  I then submitted this to
Amazon, in compliance with option (2).  Amazon promptly accepted the application and
listed U2F Zero.

After a polybag and labeler packaging party with help from my roommates,
U2F Zero is [available on Amazon](https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6).


<a href="https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6" target="_blank" style="border-bottom:none;">
  <img src="/assets/images/u2f/amazon.png">
</a>
<!--![](/assets/images/u2f/amazon.png)-->

Fortunately I had "U2F Zero" printed on the PCB before I even knew that I needed to.  Amazon may not have accepted
it otherwise.  The domain [u2fzero.com](https://u2fzero.com) points to a launch
page required for the Amazon application.  

My roommates make fun of me for how
much I order things online -- particularly from Amazon.  One of them made this [static
site](http://conorco.tech/) for me during this project (be sure to click something).

### Some costs

Including all parts, PCBs, and assembling, each token costs about $3.40 to make.  This is
cheaper then just making 1 token because costs go down when purchasing large volumes.

There's also the [cost of prototyping]({% post_url 2016-03-13-my-experience-with-dirtypcbscom %}), buying new tools,
and frictional costs.  For example, I got hit with a $126 tariff when importing the assembled
boards from China.  I also have some extra tokens not being sold to use for replacements.
I estimate the real cost was actually about $4.18 per token.

Amazon takes about $2 out of each sale.  How much Amazon takes depends on the product.  U2F Zero
qualifies for Amazon's [Small and Light](https://services.amazon.com/fulfillment-by-amazon/small-and-light.htm)
fulfillment program which has the smallest fees.

If I sell out at $8 per token, I would make a 43% ROI.  That's optimistic though.
I may have to drop the price later on if needed.  The break even price would be around $6.20.
I don't know what to expect.  I'll make a post about this later on.


# Wrap up

I'm by no means an entrepreneur but I think I'd like to keep trying to be one.
This project has been a long term experiment and the
experience has been great for me.  I am not actually concerned with financial success or growth.  The nice
thing about this project is I can just let it sit and I don't need to maintain
anything -- leaving me time to move on to the next project.

Feel free to [make your own U2F
Zero](https://github.com/conorpp/u2f-zero/wiki/Building-a-U2F-Token) or [mass
produce
it](https://github.com/conorpp/u2f-zero/wiki/DIY-Production-Programming).  Maybe
you can figure out how to produce them cheaper than what I could.


###### \* Calling something secure isn't simple.  Here's a [better analysis](https://github.com/conorpp/u2f-zero/wiki/Security-Overview).

###### \*\* Finding a chip that has secure public key crypto (P256) implementations is non-trivial.  I got lucky when I found Atmel's chip which was easy to purchase and get documentation for.  Other manufacturers that offer potential secure chips are NXP, ST Electronics, and Infineon.  But none of them sell their secure chipsets on normal distributors and seem to require customers to go through NDAs, licensing, and large minimum order requirements.  I hope the market for public key hardware becomes more transparent.

###### \*\*\* The public key in the build is signed for device attestation.



{% include JB/mytwitter %}
