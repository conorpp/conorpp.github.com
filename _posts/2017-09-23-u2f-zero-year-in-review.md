---
layout: post
title: "U2F Zero year in review"
description: ""
category: 
tags: [Startup]
image: /assets/images/default.jpg
---
{% include JB/setup %}

I wanted to share some details on the U2F Project now that it has been on the
market for over a year now.  This is to give an idea of what to expect in terms
of cost and profit from a "DIY-to-market" kind of project like this.

For those unfamiliar, [U2F Zero](https://github.com/conorpp/u2f-zero) is an
open-source secure USB device used for two-factor authentication.  I made it
back in 2016 and did a [semi-large PCB-A run and sold them on
Amazon](https://conorpp.com/designing-and-producing-2fa-tokens-to-sell-on-amazon).

At the time, I didn't really know what to expect.  Were people actually going
to buy these?  Was I going to get bad reviews?  Was someone going to find some
security bug in my open source implementation?  Is the profit worth the effort?
Now I can answer all of these questions.

# Starting out

I started on U2F Zero because I wanted to work on something interesting and
learn how to make a PCB.  Here was my first one.

![first PCB](/assets/images/pcbs/r1-front-dirty.jpg.small.jpg)

It's poorly laid out.  The traces don't give a crap about EMF.  And what's a
ground plane?

Eventually, I got better at it.  The project even got an awesome [pull
request](https://github.com/conorpp/u2f-zero/pull/37) from [Chris
Pavlina](https://github.com/cpavlina), who is better at layout than I am.
Thanks Chris! His layout is still being used today.

# Business plan

Eventually, I got the U2F Zero "minimum viable product," and it was ready to
hit the market.

![](/assets/images/u2f-zero-2/proshot.jpg)

I basically guessed that the market would sustain 1000/units per year.  So I
put in an order of 1100 units to a Chinese PCBA fab
([PCBCart](https://www.pcbcart.com/)) totaling about $4,000.  The main parts of
the cost is the PCB (~$0.22), the assembly (~$0.44) and the components
(~$2.26).  All the costs scale with quantity.  Had I ordered 3000 units, I
would have saved about $0.35 for each U2F Zero.  But the last thing you want is
2000 U2F Zeros that no one is buying sitting in a warehouse somewhere
collecting Amazon storage fees.

I didn't want to deal with sales and distribution.  I was in the middle of grad
school and didn't have the time or the interest.  So after manufacturing the
tokens, I shipped them to Amazon to be fulfilled.  This
does come with a cost, of about $1.02-$2.02 in fees per U2F Token.

After manufacturing and distribution was handled, it was time to execute my
elaborate market entry plan: write a blog post and post it to Hacker News and
Twitter.

# Sales

I spent almost a whole weekend drafting my [U2F Zero
post](https://conorpp.com/designing-and-producing-2fa-tokens-to-sell-on-amazon).
I posted it... and it worked.

![](/assets/images/u2f-zero-2/2016_sales.png)

For the first month, the time between the Amazon shipment and the post, there
was almost no sales.  Then after the post, there was a spike of *207 sales*
over the following week.  Then it balanced out to about 6-7 units per day.  And
4 months later, they sold out.

There was even an increase in sales around Christmas time.

![](/assets/images/u2f-zero-2/reviews.png)

And people left reviews.  Mostly good reviews, too.  I wasn't sure if people
would leave good reviews since it's just a cheap token with no case.  But it's
very clear from the listing that is what it is.  So most people will know what
they are purchasing -- and as long as it does a good job fulfilling that image
-- it has the potential for good reviews.  I think that was an important
takeaway.


I did get some negative feedback.  Some customers didn't like that Firefox didn't support
U2F at the time or that the token didn't come with instructions.

There was one [technical review that tore U2F Zero to
pieces](https://www.amazon.com/gp/customer-reviews/R2T5OCAPAFMO2/ref=cm_cr_arp_d_viewpnt).
Amongst the issues the reviewer found, there was privacy issue in my
implementation.  Not a critical security flaw, but something that would allow
each U2F Zero to be uniquely identified when it is used.  It was my mistake and
there was roughly 1000 units out there with it.  I ended up fixing it and am
offering replacements for those that purchased an old token.

The reviewer kindly took to opening issues on Github and even updated his
review after the firmware issues were fixed.

# Profits

As mentioned previously, each token cost about $3.00 to make.  But when
accounting for the cost of prototyping, packaging, labeling, import taxes,
tools, and holding 100 units in stock, the unit cost is really about $4.18.
There was about $2.02 in fees from Amazon.  The listing price was $8.00.  So
that works out to be roughly $2k in profit over 4 months which is about a 42%
ROI.  Not a drop-out-of school kind of haul by any means, but it was a great
experience for me and I hope to make similar ventures in the future.

For those interested, I'll list the various costs here.

* PCBs: $351.10
* Assembly + components: $3,062.85
* Shipping: $59.72
* Tariffs (from China to U.S.): $126.32
* Labels: $13.98
* Polybags: $63.25
* Tools: $164.94
* Prototyping: ~$350.00

# Other opportunities and events

There were some cool unexpected things that happened as a result of U2F Zero.
A lot of awesome people reached out or shared how they made their own U2F Zero(s) or
purchased them on Amazon.

![](/assets/images/u2f-zero-2/twitter.png)

One of which was [Landon
Greer](https://twitter.com/land0ngreer), who helped organize an infosec conference
and made custom U2F Zeros as a conference badge.
Later, the [Crypto & Privacy Village](https://cryptovillage.org/) at Defcon
forked the U2F Zero design and included it in their unofficial 2016 defcon
badge.

![](/assets/images/u2f-zero-2/acpe.jpg)

More recently, the Association for Computers Professionals in Education (ACPE)
made a custom order for 400 units and invited me to come spend some time in
Oregon and speak at their conference.  I really enjoyed it.  It was great to
learn about a different field and meet new people.

# U2F Zero, round 2

A few months after selling out, I decided to do another run.  This time I added
some improvements to the PCB layout and fix issues in the firmware.  I also
decided to order 3000 this time (saved that $0.35 per unit) and support markets
in Canada, U.K., and Europe (not just the U.S.).  I think the U2F standard is
picking up in popularity so the U2F demand shouldn't go away.  I'm curious to
see how U2F Zero sales differ between different countries.

![](/assets/images/u2f-zero-2/box.jpg)

At the time of this writing, I've sent 1500 units to the [U.S.
marketplace](https://www.amazon.com/U2F-Zero/dp/B01L9DUPK6), 300 to
[Canada](https://www.amazon.ca/U2F-Zero/dp/B01L9DUPK6), 300 to the
[U.K.](https://www.amazon.co.uk/dp/B01L9DUPK6), and 100 to a distributor in
Switzerland that will handle shipments to other countries in Europe.  I've kept
the rest in storage so I can replenish a marketplace easily if inventory gets
low.

U2F Zero has been back on the U.S. market for about $-n- weeks now, after being
out of stock for about -n- months.  Sales slowly picked back up and average
about 5 units per day again.  Pretty stable so far.

My goal is to keep the supply up with minimal effort and not have anymore
gaping out-of-stock periods.  In the meantime, I can work on the next project.

# Thanks!

Thank you to everyone that has taken an interest in U2F Zero and to those that
have contributed to the open source project so far:

* [Greg Ose](https://github.com/gregose)
* [Jean-Philippe Ouellet](https://github.com/jpouellet)
* [Chris Pavlina](https://github.com/cpavlina)
* [Eric Hahn](https://github.com/erichahn)
* [Wang Kang](https://github.com/scateu)
* [Adam Dobrawy](https://github.com/ad-m)
* [Matej Nemček](https://github.com/yangwao)
* [Landon Greer](https://github.com/land0ngreer)
* [Thomas Waldmann](https://github.com/ThomasWaldmann)
* [Robert Quattlebaum](https://github.com/darconeous)
* [Lucas Teske](https://github.com/racerxdl)
* [Cristian Rodríguez](https://github.com/crrodriguez)
* [raoulh](https://github.com/raoulh)

Thank you to everyone that supported U2F Zero and made this possible.  I've
learned a lot since first making this crappy first prototype and will have more
projects to launch later.

![](/assets/images/u2f-zero-2/early_proto.jpg)

{% include JB/mytwitter %}
