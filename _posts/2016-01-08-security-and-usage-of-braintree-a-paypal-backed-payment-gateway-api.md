---
layout: post
title: "Security and usage of Braintree, a PayPal backed payment gateway API"
description: ""
category: 
tags: [Design]
image: /assets/images/braintree/cover.png
---

{% include JB/setup %}

# My set up with Braintree

After starting an LLC, I decided I would try setting up [Braintree](https://www.braintreepayments.com/).  It's a payment service (like Stripe) owned by Paypal.
Braintree allows businesses to easily collect money from customers for a variety of payment options (Visa, Mastercard, Discover, etc) and be
[PCI compliant](https://articles.braintreepayments.com/reference/security/pci-compliance). Since Braintree
handles this and provides an interface to transfer collected money to a business account, they are considered
to be a [payment gateway](https://en.wikipedia.org/wiki/Payment_gateway).

Providing a payment gateway service is not easy from a security perspective.
The creators of the PCI standard (large credit card brands) want to minimize credit fraud and won't accept
transactions from non-compliant services.
The PCI standard is partly designed to make sure that gateway services
prevent errors from the user, where a user is a business.

So a gateway service should assume that users are oblivious.  They should assume that 
once users get a hold on some sensitive information, say some credit cards or bank account numbers, they will store it insecurely and
it will be compromised.  

In this post I will go over Braintree's security design and the infrastructure I decided to use to integrate it with my website.

# Braintree work flow

General work flow for a business is as follows:

* Generate a unique, signed identifier to represent a customer, called a client token.
* The customer will enter payment information into the Braintree origin.
* Upon submitting, Braintree will provide the business with another signed identifier representing the transaction, called a nonce (number used once).
* The business has 24 hours to use the nonce to charge the customer for money.

Check out the [docs](https://developers.braintreepayments.com/start/overview) to learn more.

## Generation of the client token

Generation of the token is provided by Braintree's server SDK.  It involves making a call to Braintree's back-end to generate a new token.
This token is then used client side to request Braintree's client SDK to create a valid payment form with Braintree as an origin.

This token is necessary for a few reasons.  It allows Braintree to create payment gateways only for authorized businesses and allows
Braintree to keep state on customers.  Outsiders won't be able to submit arbitrary purchase forms since they cannot
forge a signed identifier.


## Entering payment information

The payment form lives on Braintree's domain.  This is so that they have control over it no third parties
will have access.  So on a browser, the form will live in an iframe targeted at a Braintree domain.  The business will not be able to 
read any information from inside the iframe because of the [Same-origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy).
Braintree must do this to be PCI compliant.  

It's important to design the UI correctly so it's always clear to the user
that:

* A trusted party is handling the information
* It's clear which inputs receive payment information.

It would be unfortunate for a client to enter a credit card number in the wrong form as it could allow the business
to read the information in violation of PCI.

## Nonce representing the transaction

If the business cannot see the payment information, it needs another way to control how it charges customers.  This is 
where Braintree uses a nonce (number used once) as an identifier for a transaction.

After a user submits a valid payment, Braintree will store the potential transaction and return a nonce to the business to use as
a handle to the transaction.

Now the Nonce is just another signed token, like the client token.  But while the client token may be reused for some applications,
the nonce can only be used *once*.  This prevents duplicate charges from occurring (a good measure against those oblivious users) as 
well as sensitive information leakage later on.

The nonce should be used on the business' server to securely act on the transaction.

## Charging the customer

Presumably, the nonce can only be acted on by a server with the right API key.  The server will have access to the API
key and can charge a monetary amount to the transaction using the nonce.

Here is a python example from Braintree docs:

```python
result = braintree.Transaction.sale({
     "amount": "10.00",
    "payment_method_nonce": nonce_from_the_client
})
```


# Setting up a payment system securely

I'd say Braintree is pretty user proof.  But let's not get greedy and not set up a week infrastructure!

Let's assume our infrastructure just needs to support two services: a web application and a payment server.

If I were at a hackathon, I would probably just run the payment service and web app both on the same machine as root.  But that's a disaster waiting to happen.

You should assume that at some point, some application will get compromised. It could be because of an admin mistake or a new vulnerability discovery.  It happens.
It's important that all applications remain as separate from each other as possible.  I want my web app to be able to use my payment system
to get paid, but at the same time I don't want my payment system and account to get exploited the moment my web app inevitably gets owned.  I would like
there to be some additional difficulty.  This is the basis of [privilege separation](https://en.wikipedia.org/wiki/Privilege_separation).

It really doesn't matter that one of the services is a payment service: this decision processs should happen for every service that is distinct
and independent.

A basic improvement could be to just run the services as different users on a typical Linux or *Bsd system.  Then an attacker would have to gain priviledge
escalation after compromising the web app to exploit the payment system.  But that's still a little close.  Since the payment system isn't tied to
the performance of the web application, why not just run it on a different machine?

Here is the solution I took.

![](/assets/images/braintree/braintreesetup.svg)

I have one preexisting digital ocean instance for my blog (web app) and I added another one to run the Braintree system.
I set up another [Let's Encrypt]({% post_url 2015-11-04-trying-out-lets-encrypt %}) cert on my Braintree instance so the web app can securely proxy to the payment service via TLS.
For good measure, I set up the HTTP server on the braintree instance to be restricted with a password so only the webapp instance can access it directly.


# The result

Here my Braintree payment form.

<iframe style="height: 22em; border: 1px solid #E6E6E6;" width="100%" src="{{site.braintree}}"></iframe>

It's live and if you see check the page source, the payment info is indeed handled by a Braintree domain.  Feel free to try it out:
you will be charged $0.10 and I am happy to refund.

As usual, you can see the source for my projects on Github.

* [Braintree service](https://github.com/conorpp/braintree-box).

Let me know if you see any problems or have ideas/suggestions.

{% include JB/mytwitter %}

