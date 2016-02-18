---
layout: post
title: "Keyak: a candidate for the authenticated encryption standard"
description: ""
category: 
tags: [Crypto]
image: /assets/images/default.jpg
---
{% include JB/setup %}

# New authentication cipher

Keyak is an authenticated crypto system that is based on the Keccak sponge primitive. 
It is a candidate for the authenticated encryption standard in 
[NIST's Caesar competition](https://competitions.cr.yp.to/caesar.html).
For those not familiar, Keccak is the recent new standard for a hashing algorithm, SHA-3.
The sponge primitive is a neat construction.  By itself, it is a secure hashing primitive, 
but can easily be extended to provide encryption, pseudo random number generation, 
and authentication primitives.

In this post, I will explain how Keyak works by explaining how one can build upon sponge primitive.
Then I will share how my research group is planning to optimize it on GPU's.

# So what is an authenticated cipher again?

Not surprisingly, it's a crypto algorithm that will provide confidentiality *and* authentication to
communicating parties.  AES (the current encryption standard) only
provides confidentiality; it protects from  eavesdropping but provides no mechanism to ensure you are communicating with a friend or potential foe.

# Why not just keep using HMAC's?

An HMAC (Hash-based message authentication code) will provide authentication but that relies on hash algorithms
separate from the cipher being used.  Many different ciphers and different hashes can be used and combined to provide
authenticated ciphers, each with different trade-offs.

Not only is using two algorithms combined relatively inefficient, but because of
the lack for a standard, unnecessary complications may arise.  TLS, for example, is quite complicated and has had a
[fair share of attacks](https://en.wikipedia.org/wiki/Transport_Layer_Security#Security), many of which can be attributed
to TLS's complicated nature and wide cipher suite support.

# How Keccak comes into play

To start, let's make our own cipher that is based on Keccak and develop it until it is like Keyak.

The Keccak sponge construction is illustrated below.  All you need to understand is that it 
absorbs up to 1600 bits at a time and outputs a 1600 bit permutation.  Any additional input
can be XOR'd into the output block and fed back into the permutation round.  Rinse and repeat. 

![](/assets/images/keyak/duplex.png)

The permutation is repeated 4 times in the "absorbing" for the input and 2 times in the "squeezing" state
in the figure.  (a 6400 bit message, M, is input and a 4800 bit hash, Z, is output).


Keccak specifically parameterizes how many rounds it should go through in each `f` and then how much
output should be pulled at the end for the final hash.  But for our purposes, it isn't necessary
to think about parameters.  Let's implement our cipher.

Instead of feeding input to `f`, we can first input a *secret key* 
(and [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce)).  I.e. we
initialize the sponge state with our secret key.  This will ensure all of our following applications of `f`
are unique and full of entropy.

Now think that for each `f` of the sponge we apply, we will get a new "hashed" version of our secret key in a way.
We get 1600 bits of new "hash" each time and we can do this as much as we want.  It is these 1600 blocks
of "hash" that we can XOR our message with.  It's like a pseudo one time pad.

Our recipient, who has the same secret key (and nonce), can compute the same "hash-like" blocks.  Upon receiving
the ciphertext, he can XOR it with his computed "hash-like" blocks to get the clear text message.  

Pretty nice, huh?  The security of the cipher is based on the security of Keccak, which we can
take some comfort in.

## Adding simple and strong authentication

We're still missing the big picture: *authentication*.  Our cipher currently
only yields confidentiality.  Let's add a couple more steps.

So instead of just reapplying `f` to the secret key to get a bunch of "hash-like" blocks,
we will include the message XOR'd in each block before reapply the sponge permutation to get another block.
That way each new block is based on the initial secret key *and* the message being enciphered.  After the whole
message has been encrypted, we can run the sponge permutation one more time to get a block that we will simply
use as our *authentication tag*.

![](/assets/images/keyak/fullduplex.png)

In the figure, K is our secret key and α0 is the nonce and padding.  Cipher blocks (Z0 XOR α1), (Z1 XOR α2), ... are
output as ciphertext for all input α.

Our recipient, upon receiving the cipher text and authentication tag, still uses the secret key to compute the initial 
"hash-like" block and XOR's it with the first ciphertext block and gets the first block of plaintext.
That initial clear text XOR'd with the "hash-like" block is input to `f` to get the next block for decryption.
This is repeated until all ciphertext is decrypted.  Now for the exciting part.

The receiver can then apply `f` one more time to get a final "hash-like" block which should be a recomputed
authentication tag.  If the recomputed
authentication tag is the same as the received authentication tag, then the message is authentic!  

This is the basis for Keyak.  I find it to be quite clever and superior to current authenticate crypto systems.  Not only
is it based on Keccak, but it combines encryption and authentication into one innate process.  This sure beats the canonical
method of encrypting and then afterwards calculating an HMAC over it.

# Get prepared for creativity

There is certainly more to Keyak then the simple process I described.  It supports accepting *meta-data*
which can be sent as cleartext but will be included in the sponge permutation input so that it is
still authenticated.  It can be parameterized to set the block size, Keccak permutation used,
and number of encryption processes to run in *parallel*.  Yup, Keyak supports parallelism as well.

![](/assets/images/keyak/keyak_block_s.png)

My impression is that Keyak is modeled after a motor boat.  It has the following layers:

* **Motorist layer**: responsible for handling the parameters.
The motorist starts an "engine" which can be fed input (either ciphertext or plaintext) to be wrapped, where
wrapping is either encrypting or decrypting.  If decrypting, an authentication tag will also be input and 
verified.  If the verification is successful, the engine will return the plaintext or it will fail.  If encrypting,
an authentication tag and ciphertext will be output.

* **Engine layer**: responsible for distributing blocks of input to it's various pistons and piecing
together the piston output.  

* **Piston Layer**: where the work actually gets done.
Each piston can be initialized with a secret key and nonce and
they apply `f` to a particular input to update the sponge state with a new "hash-like" block.
They do the wrapping of the input with the "hash-like" blocks to get the ciphertext or plaintext
as in our simple algorithm above.

The pistons do the bulk of the work and each one is independent of the other.  This allows them to be run in parallel
for performance.

# Optimizing Keyak for performance

For a naïve optimization, a Keyak could be parallelized using thread level parallelism by simply assigning
a thread for each piston.  It's a little overkill since each thread would be reduntantly executing the same
instructions, just on different data.  

Let's consider Flynn's Taxonomy which contains 4 classifications for
computer architectures.

![](/assets/images/keyak/flynn.jpg)

* **SISD** (single instruction single data) where 1 instruction is computed at a time, each
on a different data.  Your regular CPU cores do this.

* **MISD** has multiple instructions executing in parallel for the same piece of data.  This
is not common but could be useful for fault tolerance.

* **SIMD** has one instruction execute for multiple pieces of data.  This is common in graphics
processing and is abundant in GPUs.

* **MIMD** is multiple instructions executing in parallel for different pieces of data.  Modern CPUs do
this by having multiple cores.

So pistons in Keyak are each running the same instructions, but each on different data.  Any idea out which 
architecture would be best for optimizing Keyak?

If you think it's SIMD (single instruction multiple data), then you are right!

My research group will be looking into using SIMD (Single Instruction Multiple Data) to optimize Keyak.
SIMD consists of running the same set of instructions on different data paths which is pretty useful
and commonplace in graphics so we will be implementating Keyak optimizations on a GPU.

# Wrapping it up (pun intended)

Keyak is a great study because not only is it a good read for it's overarching Engine metaphor, but
it's a completely different way to do encryption from an algorithmic standpoint.  It's totally different from the typical 
substitution-permutation networks that many current ciphers are based on, including AES.  It's kind of weird, 
really; but that's likely a good thing for a contender in the CAESAR competition.  

This is a high level post; to learn more of the details, check the [Keyak website](http://keyak.noekeon.org/)
and the paper below.


### References

* [Wetzels J., Bokslag W., "Sponges and Engines An Introduction to Keccak and Keyak", 2016](https://eprint.iacr.org/2016/028.pdf)

* Thanks to [Bilgiday Yuce](http://rijndael.ece.vt.edu/bilgiday/index.html) for the Motorist figure.

{% include JB/mytwitter %}
