---
layout: post
title: "My experience with DirtyPCBS.com"
description: ""
category: 
tags: [Hardware]
image: /assets/images/pcbs/box-2.jpg
---
{% include JB/setup %}


# Dirty Cheap Dirty Boards

*(warning ~30 MB of photos)*.

If you want to order custom made PCB's for a small price, [Dirty PCB's]() is
probably your cheapest option.  This used to just be a service that was internal
to [Dangerous Prototypes](http://dangerousprototypes.com/) to make cheap boards
for their own projects.  Now it's public and they make dirty cheap boards for anyone.

To get an idea, you can buy ~10 boards that are 5x5 cm for $14 (free shipping).
You can get more boards, larger sizes, and more layers as well for relatively cheap prices.

Here I'll share my experience with Dirty PCB's and photos of a [U2F 2FA token](https://fidoalliance.org/about/overview/) I've been working on.  I ordered
the same boards from Seeed Studio as well to compare.

# The revisions

This is my first PCB so the first one is pretty dirty by design and not just because of Dirty PCB's.
The next revision is a better layout.

## Revision 1 from Dirty PCB's


![R1 Dirty PCB's front](/assets/images/pcbs/r1-front-dirty.jpg)


![R1 Dirty PCB's back](/assets/images/pcbs/r1-back-dirty.jpg)


## Revision 1 from Seeed Studio



![R1 Dirty PCB's front](/assets/images/pcbs/r1-front-seeed.jpg)


![R1 Dirty PCB's back](/assets/images/pcbs/r1-back-seeed.jpg)


You can see that the layout is pretty poor.  None of the traces really make 45 degree angles
or care about what angle they cross over each other between layers.

Also there is no ground plane.  I don't really need one but decided to add one later.

There isn't too much of a difference in quality between the two boards.
But you can still see why Dirty PCB's is cheap:

* The silkscreen (white coloring) is smudgy and doesn't have as good of a tolerance.
* The solder mask is not as well aligned with the pins.  Look at where the red/green covering meets the pins.
* The vias are bigger and make some of the pins look smudged.

Look at both board pictures in new tabs and you can see these differences in quality for yourself.


At the time Seeed was a little more expensive ($10 for 10 boards + $10 shipping) and charged more for 2mm thick
boards which is necessary for me because they fit best in a USB socket.  Now it seems [their pricing](http://www.seeedstudio.com/service/index.php?r=pcb) is similar
to [Dirty PCB's](http://dirtypcbs.com/index.php).

After revising my layout design, I made two more orders from Dirty PCB's.

## Revision 2 Dirty PCB's


![R2 back](/assets/images/pcbs/r2-front.jpg)


![R2 back](/assets/images/pcbs/r2-back.jpg)

I'm not sure what happened to the silkscreen on this one.  It seems any traces or ground plane overlapped 
the silkscreen.  I suspect it was an issue at the board house since the next revision, (which is really the same layout, just a different color) did not suffer
this problem.


## Revision 3 Dirty PCB's


![R3 back](/assets/images/pcbs/r3-font.jpg)

![R3 back](/assets/images/pcbs/r3-back.jpg)

# Where to get PCB's

Overall, small PCBs are pretty cheap.  I think Seeed will offer slightly higher quality for a similar price.  Dirty PCB's will
charge less for other options like color and stencils and will be quicker out of the board house.

If you PCB design isn't crazy on tolerances you can't go wrong with either one.

# ** Update

Based on replies I've gotten, here are some other places to check out for maxing out cheapness and quality:

* [Hackvana](http://www.hackvana.com/store/)
* [Elecrow](http://www.elecrow.com/services-pcb-prototyping-c-73_116.html)
    * 10 PCBs for $9.9, 4 to 5 days production, $18 stencil

Check out [PCBshopper.com](http://pcbshopper.com/) to query the price of most fabs for a given board.


{% include JB/mytwitter %}
