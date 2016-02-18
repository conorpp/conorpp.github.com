---
layout: post
title: "Keyak: a candidate for the authenticated encryption standard"
description: ""
category: Crypto
tags: []
image: /assets/images/default.jpg
---
{% include JB/setup %}

# New Authentication Cipher

Keyak is an authenticated crypto system that is based on the Keccak sponge primitive. 
It is a candidate for an authenticated encryption in NIST's Caesar competition.
For those not familiar, Keccak is the recent new standard for hashing, SHA-3.  The sponge primitive is a neat construction.  By itself,
it is a secure hashing primitive, but can easily be extended to provide encryption, pseudo random number generation, 
and authentication primitives.

In this post, I will go over how Keyak works and how it uses the sponge primitive.

# So what is an authenticated cipher again?

Surprisingly, it's a crypto algorithm that will provide confidentiality *and* authentication to
communicating parties.  AES (the current encryption standard), on the other hand, will only
provide confidentiality; no one can eavesdrop but it provides no mechanism to ensure you are communicating with a friend or foe.

# Why not just keep using HMAC's?

An HMAC (Hash-based message authentication code) will provide authentication but that relies on hash algorithms
separate from the cipher being used.  Many different ciphers and different hashes can be used and combined.  Because of
the lack of a standard, unnecessary complication arises.  TLS, for example, is quite complicated and has had a
[fair share of attacks](https://en.wikipedia.org/wiki/Transport_Layer_Security#Security), many of which can be attributed
to TLS's complicated nature and wide cipher suite support.

# How Keccak comes into play

To start, let's make our own cipher that is based on Keccak and develop it until it is Keyak.

The Keccak sponge construction is illustrated below.  All you need to understand is that it 
absorbs up to 1600 bits at a time and outputs a 1600 bit permutation.  Any additional input
can be XOR'd into the output block and fed back into the permutation round.  Rinse and repeat.

![](/assets/images/keyak/duplex.png)

Keccak specifically parameterizes how many rounds it should go through and then how much
output should be pulled at the end for the final hash.  But for our purposes, it isn't necessary
to think about parameters.  Let's implement our cipher.

Instead of feeding input to the permutation, we can first input a *secret key* (and [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce)).  In other words, we
initialize the sponge state with our secret key.  This will ensure all of our permutations rounds
are unique and full of entropy.

Now for each round of the sponge we apply, we will get a new "hashed" version of our secret key in a way.
We get 1600 bits of "hash" each time and we can do this as much as we want.  It is these 1600 blocks
of "hash" that we can XOR our message with.  It's like a pseudo one time pad.

Our recipient, who has the same secret key (and nonce), can compute the same "hash-like" blocks.  Upon receiving
the ciphertext, he can XOR it with his computed "hash-like" blocks to get the clear text message.  

Pretty nice, huh?  The security of the cipher is based fully on the security of Keccak, which we can
take some comfort in.

## Adding simple and strong authentication

We're still missing the big picture: *authentication*.  Our cipher currently
only yields confidentiality.  Let's add a couple more steps.

So instead of just reapplying the permutation to the secret key to get a bunch of "hash-like" blocks,
we will include the message XOR'd in each block before reapply the sponge permutation to get another block.
That way each new block is based on the initial secret key *and* the message being enciphered.  After the whole
message has been encrypted, we can run the sponge permutation one more time to get a block that we will simply
use as our *authentication tag*.

Our recipient, upon receiving the cipher text and authentication tag, can compute the initial "hash-like" block to XOR and get the first
1600 bits of clear text.  That 1600 bits of clear text XOR'd with the "hash-like" block is input to the sponge permutation
to get the next block for decryption.  This is repeated until all ciphertext is decrypted.  Now for the exciting part.

The receiver can then apply the sponge permutation one more time to recompute the authentication tag.  If the recomputed
authentication tag is the same as the received authentication tag, then the message is authentic!  

This is the basis for Keyak.  I find it to be quite clever and superior to current authenticate crypto systems.  Not only
is it based on Keccak, but it combines encryption and authentication into one innate process.  This sure beats the canonical
method of encrypting and then afterwards calculating an HMAC over it.

# Get prepared for creativity

There is certainly more to Keyak then the simple process I described.  It supports accepting *meta-data*
which will be data sent in the clear but included in the sponge permutation input so that it is
still authenticated.  It can be parameterized to set the block size, type of Keccak permutation used,
and number of encryption processes to run in parallel.  Yes, Keyak support parallelism as well.

Keyak runs in a mode called "motorist" mode.  The motorist is responsible for handling the parameters.
The motorist starts an "engine" which can be fed input (either ciphertext or plaintext) to be wrapped, where
wrapping is either encrypting or decrypting.  If decrypting, an authentication tag will also be input and 
verified.  If the verification is successful, the engine will return the plaintext or it will fail.

The engine is responible for distributing blocks of input to it's various pistons.  The pistons are
the components actually doing the work here.  They can be initialized with a secret key and nonce and
they apply the sponge permutation to particular input to update the sponge state with a new "hash-like" block.
Finally, they also do the wrapping of the input with the "hash-like" blocks to get the ciphertext or plaintext
as in our composed algorithm above.

The pistons do the bulk of the work and each one is independent of the other.  This allows them to be run in parallel
for performance.  The keyak authors currently recommend using one piston but I hope someone pushes for more than
one piston in the standard to improve overall performance.

# Wrapping it up (pun intended)

Keyak is a great study because not only is it a good read for it's overarching Engine metaphor, but
it's a completely different way to do encryption from an algorithmic standpoint.  It's totally different from the typical substitution-permutation
networks that many current ciphers are based on, including AES.  It's kind of weird, really; but that's a good thing 
for a contender in the CAESAR competition.




{% include JB/mytwitter %}
