---
layout: post
title: "Cloudflare Wallets: Giving Agents Identity and Money"
date: 2026-08-10
---

On August 4, Cloudflare announced **Cloudflare Wallets**, programmable wallets designed to give AI agents both a stable identity and the ability to spend money.

[Cloudflare Wallets announcement](https://blog.cloudflare.com/wallets/){:target="_blank" rel="noopener noreferrer"}

The interesting part isn't the wallet.

It's the primitive underneath it.

An agent needs to act on your behalf. That means it needs **identity, authority, and money**.

The model is simple:

**Human → Agent → Wallet → Internet service**

The human sets the rules. The agent gets a limited budget. The service sends a payment request. The agent pays and gets access.

Cloudflare calls these **Account Wallets** and **Virtual Wallets**. Virtual Wallets can have spending limits, allowlists, and maximum transaction sizes. They give an agent a budget without giving it the keys to your bank account.

The identity piece may be just as important.

Cloudflare is introducing **cloudflare.pay** handles so an agent can have a persistent, human readable identity. For example:

`research.example.cloudflare.pay`

That lets a merchant know that an agent is operating on behalf of a particular account or organization.

You can already reserve a handle at [cloudflare.pay](https://cloudflare.pay/){:target="_blank" rel="noopener noreferrer"}. Reservations are free, limited to one per Cloudflare account, and currently create a simple page at `HANDLE.cloudflare.pay`.

The exciting part is what this enables.

Imagine your research agent discovering an API it has never used.

Instead of asking you to create an account, enter a credit card, generate an API key, and approve the purchase, it can simply use its budget.

A few cents here.

A dollar there.

A new service when it needs one.

The agent can **explore the Internet economically**.

That's a fundamentally different model from today's web.

Cloudflare isn't just building a crypto wallet for AI.

It's building a primitive for **delegated economic agency**.

And if wallets, identity, and machine readable services converge, the Internet starts looking less like a collection of websites and more like a marketplace where software can discover, evaluate, and buy capabilities on our behalf.

That is the exciting part.
