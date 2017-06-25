---
layout: post
title: "Last post on emulating a credit card"
description: ""
category: Design
tags: [Design, Fail]
image: /assets/images/default.jpg
---
{% include JB/setup %}

This is an update to a previous post about designing a [credit card
emulator](/designing-a-credit-card-emulator-card).  I got a custom made coil
and it didn't work.  I'm concluding the project here.


I first attempted to generate a magnetic field using only PCB traces as this
would be easy and cost effective to create.  But the field was not strong
enough to be read by an external reader.  I then wound my own coil with 36
gauge magnet wire such that it had a much higher coil density while still being
as small as a mag-strip track.  This *worked* but only when I held the card
still in the reader -- if I moved the card while sending a signal (emulating a
swipe), the reader would often fail to read the signal sent.  I hypothesized
that since my coil was crudely hand wound using a power drill, it had slightly
non-uniform windings which caused significant variations in magnetic field as
the non-uniform coil moved past the reader.

What I should have done at this point was invest some time in making a setup to
read the full signal on an oscilloscope and experiment further.  I was feeling
somewhat impatient with this project at this point though, so I went ahead and
got a coil wound by a shop that could get much more uniform windings.

![custom coil](/assets/images/card/customcoil.jpg)

![custom coil](/assets/images/card/customcoil2.jpg)

Pictured above, is my "uniform" coil and it kapton-taped to a mag-strip
blank.  I tried it on my previous set-up with a mag-strip reader and
unfortunately it did not work.  The signal ending up being too weak, even when
driving 0.5 amps through it.  Despite the coil being uniform, now it probably
has too small of a coil density because I did not specify this well when I
ordered it.  I have lost interest in this project but I'll conclude it with
some takeaways.

**Better testing means less iteration**.  My testing environment consisted of a
[driver circuit](https://github.com/samyk/magspoof) with some test credit card
numbers and a mag-strip reader.  This basically just gave me a boolean answer
to whether if my design worked or not.  Instead of trying out many different
designs on this setup, it would have been better to put more time in improving
the test set up with an oscilloscope so I could see more, and then try out only
a couple designs.  Then by the time it comes to order a custom coil, I have a
better idea of the parameters to use.  This seems obvious in retrospect but I
think I had tunnel vision on this project while grad school demanded most of my
attention.

**The optimal coil for mag-strip emulation**.  It probably can't be done with
PCB traces unless you drive a ton of current (5+ amps), which is impractical
for credit card form factors.  It will need to fit in a box roughly the size of
3in x 0.11in x 0.07in.  I don't know any off the shelve coils you can get
that would fit those dimensions so you will likely need to custom make it.

**Synchronization**.  For emulation to work, the signal needs to be sent
through the coil when it is in range of the credit card reader, not a moment
sooner or later.  How could it possibly know when?  My idea was to use small
buttons to act as one bit pressure sensors, which would get activated when a
credit card is swiped through a reader.  I was looking into using [metal
domes](https://conorpp.com/a-review-of-some-smt-buttons) like from snaptron
which come in ideal form factors and force ratings.  I got the buttons but
didn't make enough progress to try them for this purpose.

If anyone has a use for the coils or PCBs I made, let me know.  I'm not going to use them.



{% include JB/mytwitter %}
