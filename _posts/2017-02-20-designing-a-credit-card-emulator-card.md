

# What this project is

Some time ago I got interested in emulating magstripes with electromagnets.
There are some other projects that have done this, like [magspoof](https://github.com/samyk/magspoof).
What I am really interested in figuring out is how to get the form factor to the size of a credit card.
Arguably there isn't much utility to this except maybe storing multiple magstripe cards on one card, like
some [commercial products](http://blog.onlycoin.com/) do.  But I still think it's cool and I'd like
to make it into a badge for the next DefCon.

# Design challenges and fails

A normal magstripe card is embossed with a magnetic strip which holds either 75 or 210 bits per inch normally.
You can picture it as taking a bunch of tiny magnets and laying them out with the poles pointing up and down the stripe.
When you swipe the card, all the little magnets go by the reader, and the reader sees the magnetic polarity change a bunch
of times.

![](https://i.imgur.com/a3eUAba.jpg "Image courtesy of Wikipedia.")

You can emulate it easily with an electromagnet by just changing the current direction (a magstripe
reader will see the same changes in polarity).  One thing people might initially be confused by is how the reader "clocks"
the data in.  I.e. different people swipe cards at different speeds, how does the reader synchronize the data?  The information is encoded
in [F2F](https://en.wikipedia.org/wiki/Differential_Manchester_encoding).  Basically the little magnets are sized such that the polarity will change
at two rates, where one rate is always twice that of the other rate.  So it can be synchronized by looking at the relative changes in polarity with
respect to time.  Binary one pulses will be twice as long as binary zero pulses.

### Electromagnet design

The hardest part of this project I believe is getting a proper coil and enough current to run through it so the magnetic field is strong enough.
I first tried seeing if traces on a 2 layer PCB could suffice to generate a magnetic field strong enough for a reader.

The ISO specs don't actually specify the magnetic field strength that is needed, but rather say something along the lines
of "it should be between 0.7 and 1.2 times that of a reference card."  I couldn't really theorize much so I just went ahead and tried it.

![](https://i.imgur.com/KSPqB8V.jpg)
![](https://i.imgur.com/HTYm7b7.jpg)

This is front and back of the first PCB card I tried.  I drove it using a similar circuit as in magspoof and tested it
with a real magstripe reader.

The traces are going up/down the card in the two magstripe track locations so that the magnetic field generated would be pointed up/down the stripe like in a real magstripe.  

I tried to maximize the number of turns per length in attempt to make the field stronger but I didn't think of the major field
cancellation that would occur since the traces on front/back of the card are aligned to each other.  Needless to say, it did not work.

![](https://i.imgur.com/43ExcX1.jpg)
![](https://i.imgur.com/fVIxFtd.jpg)

For this next iteration, I half fixed the field cancellation issue.  I drew the front and back traces to be at a 45 degree angle
with respect to each other.  So there would still be cancellation, but it would be half as much as before.  I figured that if using
a 2 layer PCB for this purpose is going to work at all, this was the way to draw it.  But I'm no electrical engineer; if anyone has
any other ideas, please comment or contact me.

It worked a *little* bit with the magstripe reader.  I made a crude test with magnetic filings to compare the magnetic strength of the
PCB traces with a credit card to see
if it fell within the "0.7 to 1.2" factor range.

![](https://i.imgur.com/PRSfNtE.jpg)
![](https://i.imgur.com/am9alEF.jpg)

I ran about 2 amps through the PCB traces and the amount of iron filings that stuck to the coil was dismal compared
to the credit card.  I figure the only thing that could work after this is a custom made coil.

### Next attempt

I made my own coils using 36 gauge magnet wire, plastic, tape, and a drill.  Running it through the reader, it seemed to
work pretty reliably.

![](https://i.imgur.com/nuDeFCx.jpg "I didn't wind the whole coil because lazy")

I ordered some custom shaped coils that are uniform and thin enough to fit in a credit card form factor.  I did this by
searching on Alibaba and emailing some drawings.

I will make an update later on!

### Acknowledgements

Thanks to [PCB minions](https://pcbminions.com/) for making good PCBs and having good service.
