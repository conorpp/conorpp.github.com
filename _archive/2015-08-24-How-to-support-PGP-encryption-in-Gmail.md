---
layout: post
title: "How to support PGP encryption in Gmail"
description: ""
tags: [Tools, Crypto]
image: /assets/images/default.jpg
nocover: true
archive: true
---
{% include JB/setup %}

# How to support PGP encryption in Gmail

I like to use PGP whenever I can.  Not everything I send is necessarily super sensitive
or private, but I'd prefer it to never show up in a leaked database.  Good thing
Google has been working on a Chrome extension [End-to-End](https://github.com/google/end-to-end) to add PGP support to Gmail.

**Disclaimer**:  As of the time of this posting, the extension is in the *review* stage of development.
No releases have been made.  But it's open for testing!  It's also included in Google's Bug Bounty Program.

# Setting up

I'm running Ubuntu 15.04 and I had to install Ant and Java.

```
sudo apt-get install ant openjdk-8-jdk
```

Now to get the source and built it.

``` bash
git clone https://github.com/google/end-to-end
cd end-to-end/

# Downloads the source for closure-compiler and builds it.
./do.sh install_deps

./do.sh build_extension
```

Now we can load the extension into Chrome.  Turn on developer mode and click the "Load unpacked extension" button.

![Developer mode and unpacked extension](/assets/images/devmode.png "Developer mode and unpacked extension")

Make sure to use \<path-to-end-to-end\>/build/extension as the unpacked extension.  Upon installing it
you should see a "Welcome to End-to-End" page open.  The icon should be on the top right of the window as well.


If you aren't familiar with PGP, I recommend you start with reading the [GPG Privacy Handbook](https://www.gnupg.org/gph/en/manual.html)
so you don't mess it up.  Key management is important and PGP relies on a web of trust model.  Most PGP users don't want unintelligible people
in their web of trust.

# Usage

Open a new tab and click the End-to-End icon.  Then click "options".

![The dropdown for a new tab](/assets/images/end2end/options.png)

This will bring you to a new page where you can do key management.
You can import your public key ring or any private keys you have.  

If you import a private key with a password on it, End-to-End will decrypt the
key.  It stores all private/public keys unencypted unless you protect them with a passphrase.
It also has a backup recovery code for recovering the keyring if you forget your passphrase.  I'm
not sure how that works.

It doesn't encrypt the keys in memory but they are "safe" inside Chome's sandbox mode.

![The dropdown for a new tab](/assets/images/end2end/keymng.png)


Now for writing an email.  Just go to Gmail and click on the extension icon again and a email form will show up.
You can then write an email like normal but be sure to save encrypted drafts often!  End-to-End will not do it
for you like Gmail does, and as soon as the window loses focus, End-to-End closes and all of your unsaved progress
will be gone. 

*Edit*:  This can be avoided by opening the message in a popup window (clicking the popup icon on the top right).  This
won't auto-close when the window loses focus.

![](/assets/images/end2end/message.png "The demo (John Barlow's A Declaration of the Independence of Cyberspace)")

If you have any questions about End-to-End, check out the [FAQ](https://github.com/google/end-to-end/wiki/FAQ).

{% include JB/mytwitter %}
