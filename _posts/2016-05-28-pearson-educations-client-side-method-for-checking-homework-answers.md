---
layout: post
title: "Pearson Educations client side method for checking homework answers"
description: ""
category: 
tags: [Reversing]
image: /assets/images/pearson/logo.svg
nocover: true
---
{% include JB/setup %}

# Exploitable Method

By exploitable, I don't mean in the traditional security sense.  I mean 
exploitable by lazy undergraduate students.

Pearson Education runs a large set of web applications that colleges use 
to automate their tedious ABET required courses.  I was stuck in one
my senior year and noticed that they check
homework answers client side.

Pearsons physics web application (Mastering Physics) used to be 
"exploited" via a small bit of JavaScript that would expose the homework answers loaded
client side.  It has long been fixed.  As I'll show you in this post, homework answers are still checked client side.  They
just *obfuscated* it further.

*Disclaimer*:  I don't like cheating.  I'm not providing a means for others to cheat.
<!--I'm-->
<!--just writing about how Pearson currently checks homework answers client side.-->

# Obfuscated client side checking

For the class I was in, the homework was run in a bloated Flash application.
 
![](/assets/images/pearson/flash-app.png)

I considered looking at how the applet
worked using a decompiler or debugger but quickly realized how big of a pain it would have been as
the applet is over 1 MB.  Instead, I looked at other files the applet uses.

If you open Chrome's network tab in the development tools, you can see all of the resources that
the flash applet loads.  Looking at the other resources downloaded, one file called "I0604041.Ipx" stood out.
It can be downloaded without any authentication,
so feel free to follow along:

```bash
curl -o hw.ipx 'https://www.mathxl.com/books/sullee16/Ipx/c6/I0604041.Ipx'
```

After a quick inspection, one can see that it is just a zip file.  Let's see what's inside.

```bash
unzip hw.ipx
```

Unzipping reveals two XML files: `_manifest.xml` and `14037884_006!Sullivan15e.6.4.5.tdxex`.
At this point a lazy student would be hoping to find the answered enumerated in one of the files.
But it's not that simple.

`_manifext.xml` just contains some irrelevant metadata.  The other file is quite
large (~1420 lines of XML) and details the layout and problems for the homework.  No where in it
are the "answers".


Let's look closer and try to figure out how it's used the by the Flash applet.  Here is a snippet
from the large XML file: 

```xml
<text>The PW of the difference between the old and new systems is $</text>
<control br="0" vshiftabs="1" spacing="1" textstyle="Normal" data="">S1</control>
```

It is a homework problem that asks for the PW of the difference between two systems.  It would normally be displayed
in the Flash applet.  After the "$" is a spot for the student to enter an answer.  But in the XML
we see a "control" tag instead that wraps a value called `S1`.

If we search the XML for other references to `S1`, we see something that looks like a definition:

```xml
<field name="S1" format="" tol="5" acceptunformattedvalues="1" rule="numeric">
  <solution>
    <expr>~dpw</expr>
  </solution>
```

Here `S1` gets defined as `~dpw`.  Okay.  Let's continue down the rabbit hole and see if we
can find a definition for `~dpw`.


```xml
<var name="dpw" type="int" format=",">
  <constraint>
    <incl>
      <expr>-~dcost+~dlc*((((1+0.01*~p)^60)-1)/(0.01*~p*(1+0.01*~p)^60))+~dmv/((1+0.01*~p)^60)</expr>
    </incl>
  </constraint>
</var>
```

It looks like the definition for `dpw` is a non-trivial equation that contains references to other variables like `dcost`, `dlc`, and `p`.
But the equation looks suspiciously like the work needed to be done to *honestly* solve the
problem about "PW system differences".

At this point I made a pretty confident hypothesis of how the flash application works.  It reads this XML file to 
generate the markup for all the homework problems.  It then recursively follows this definition tree to figure out what the
answers are.  It also is calculating the answers on the fly, meaning no free lunch for the lazy student.  After
solving the homework problem the honest way, I manually solved the tree of definitions and found the correct answer, which confirmed my hypothesis.

Assuming that most of Pearsons online classes use the same functionality, Pearson is clearly checking homework answers
client side, which can always be exploited.  

For this case, it would require someone to go through more work recursing through
an XML file than it would be to just do the homework.  Alternatively, someone could write a program to automate it -
but who knows how much work that would be, as it would be equivalent to rewriting Pearsons Flash applet.

# Should Pearson change anything?

Of course, this can be solved by server side checking.  But I feel it's not really necessary.  The Flash applet is hard enough
for the average student to cheat.  And in the end, cheating students only cheat themselves.



{% include JB/mytwitter %}
