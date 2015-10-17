---
layout: post
title: "How to fix a corrupted file by brute force"
description: ""
tags: [CTF]
image: /assets/images/bitflip/cover22.png
---
{% include JB/setup %}

## The Challenge

I recently competed in [CSAW](https://ctf.isis.poly.edu/) with a few friends.  For those that don't know,
[CSAW](https://ctf.isis.poly.edu/) (Cyber Security Awareness Week) is an online `CTF` where a group at NYU Poly puts up challenges related to 
computer security.

One of our favorite challenges was called "sharpturn" and hinted about SATA controller failure.  
It contained a link to a git repo. 

It contained no source files but it still had the git objects tree (typically located in .git/objects/).
We downloaded the repo and quickly checked the integrity of the files.

```bash
$ git fsck -v
```

Git complained about 3 corrupted objects.

```bash
error: sha1 mismatch 354ebf392533dce06174f9c8c093036c138935f3
error: 354ebf392533dce06174f9c8c093036c138935f3: object corrupt or missing

error: sha1 mismatch d961f81a588fcfd5e57bbea7e17ddae8a5e61333
error: d961f81a588fcfd5e57bbea7e17ddae8a5e61333: object corrupt or missing

error: sha1 mismatch f8d0839dd728cb9a723e32058dcc386070d5e3b5
error: f8d0839dd728cb9a723e32058dcc386070d5e3b5: object corrupt or missing
```

## What we did

For those that don't know, a git object is just a `zlib` compressed version
of a file that is being tracked.  It could inflate to be any file.  So we uncompressed
all of the corrupted git objects to find that they were 3 different revisions
of a C++ source file. The `f8/*` file looked like the latest revision so we looked at that.


```bash
zlib-flate -uncompress < objects/f8/d0839dd728cb9a723e32058dcc386070d5e3b5 > source.cpp
```

Looking at source.cpp, we can see that it asks the user 5 questions:

```c++
std::string part1;
cout << "Part1: Enter flag:" << endl;
cin >> part1;

int64_t part2;
cout << "Part2: Input 51337:" << endl;
cin >> part2;

std::string part3;
cout << "Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194" << endl;
cin >> part3;

std::string part4;
cout << "Part4: C.R.E.A.M. Get da _____: " << endl;
cin >> part4;

uint64_t first, second;
cout << "Part5: Input the two prime factors of the number 270031727027." << endl;
cin >> first;
cin >> second;
```

All of which are pretty simple:

1. flag
2. 51337
3. ok
4. money
5. 29, 271, 1103, 31151

Except the 5th question is clearly wrong.  There are 4 prime factors to 270031727027,
not 2.  So this is where challenge starts to become clear: SATA controller failure is hinted at,
one of the files is corrupt, and it asks a mathematically impossible question.  There must
have been bit flips in the file.

We think this even more when we try to compile the file and run into an error:

```c++
std::string flag = calculate_flag(part1, part2, part4, factor1, factor2);
cout << "flag{";
cout << &lag;
cout << "}" << endl;
```

Clearly `&lag` was meant to be `flag`.  If we look at the `ASCII` values for '&' and 'f',
we see they are `0x26` and `0x66`, respectively.  If you flip the 7th most significant bit in
`0x66`, you would get `0x26`.

To solve this challenge, my friend [Jean-Philippe](https://github.com/jpouellet) decided it would be fun to write a program
that would combinatorially flip bits on a file in the most efficient way he could come up with. 
The program loaded the corrupt file into memory and incrementally flipped bits and recalculated the 
SHA1 hash each time to see if it matched.  

If there's more than one bit flipped, this approach could take a very long time.  First it would
have to check all combinations of flipping 2 bits, then all combinations of flipping 3 bits, and so on.
The file we're looking at is 15,120 bits long.  So if there are 3 bit flips, this approach would take
`15,120 choose 3` operations in the worst case (5.76e+11).  Since that would probably take longer then the competition lasted,
we had to try it on versions of the file where we fixed the bit flips we already thought were errors.

So we changed the `&lag` error from before and also changed one question to ask for "31337" because
that's leetspeak for "eleet".

```c++
cout << "Part2: Input 31337:" << endl;
```

Running [Jean-Philippe](https://github.com/jpouellet)'s program instantly yielded the correct file after this.  Turns
out there was only one bit flip left and it was in the "270031727027" string.

Now the program asks:

```bash
Part1: Enter flag:
flag
Part2: Input 31337:
31337
Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194
ok
Part4: C.R.E.A.M. Get da _____: 
money
Part5: Input the two prime factors of the number 272031727027.
31357
8675311
flag{omitted}
```

We got the flag and got 400 points in the competition. Most of the challenges rewarded 100-500 points.

## You should try it out

I went ahead and made some modifications to the [program](https://github.com/conorpp/bitflipper) so we could share it.  I probably made the code a little uglier
but more user friendly.  You can try it out [here](https://github.com/conorpp/bitflipper).


{% include JB/mytwitter %}
