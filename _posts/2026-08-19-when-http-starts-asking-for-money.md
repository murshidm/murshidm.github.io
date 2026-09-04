---
layout: post
title: "HTTP 402 Is Having a Moment"
date: 2026-08-19
---

There is an old HTTP status code that was never really used. Now, as AI agents begin to request and pay for online services, it may finally be making a comeback.

**402 Payment Required.**

It was reserved decades ago for a future where a web request might require payment. That future never really arrived.

AI agents may finally give it a reason to exist.

An agent does not want to create an account, enter a credit card, generate an API key, and negotiate a subscription every time it discovers a useful service. It wants to request something, understand the price, pay, and get the result.

That makes HTTP 402 surprisingly interesting.

### The basic idea

The interaction is simple.

An agent requests a resource. The server responds with `402 Payment Required` and describes what payment is required. The agent pays, retries the request with a payment credential, and the server verifies it before returning the resource.

**Request → 402 → Pay → Retry → Response**

Payment becomes part of the HTTP interaction rather than a separate checkout experience.

This is the basic idea behind protocols such as **x402** and the **Machine Payments Protocol**, or MPP. [Cloudflare's agent payment infrastructure](https://developers.cloudflare.com/agents/tools/payments/){:target="_blank" rel="noopener noreferrer"} now supports both, while Stripe and Tempo are developing MPP as an open standard for machine payments. [Stripe's introduction to MPP](https://stripe.com/blog/machine-payments-protocol){:target="_blank" rel="noopener noreferrer"} explains the broader goal.

The significance of 402 is therefore less about a new payment method and more about changing what an HTTP request can mean.

A request no longer has to mean:

**Give me this resource.**

It can mean:

**Give me this resource, and tell me what it costs.**

### x402

**x402** emerged from Coinbase and takes a crypto first approach to machine payments.

A service can return a 402 response containing payment requirements. The agent pays, attaches proof of payment to its next request, and the service verifies the transaction.

The model is particularly interesting for APIs, data, content, and other resources that can be priced per request.

It also removes some of the machinery associated with traditional web payments. There does not necessarily need to be an account, subscription, or API key for every service.

The machine can simply pay for what it uses.

### MPP

**Machine Payments Protocol**, developed by Stripe and Tempo, takes a broader approach.

MPP is designed to support different payment methods, including stablecoins and traditional payment methods such as cards. It also supports different transaction patterns, including one time charges, sessions, and subscriptions.

This creates an interesting contrast.

x402 starts from the idea of putting programmable money into HTTP.

MPP starts from the idea of giving machines access to the existing payment ecosystem through an Internet native protocol.

Both are trying to solve the same emerging problem: **how does software pay software?**

### The competitive landscape

The interesting thing is that there is not one competition happening here. There are several.

Some companies are trying to own the commerce experience. Others are working on authorization, agent identity, payment protocols, payment networks, or settlement.

**Coinbase and x402** focus on machine payments and stablecoin payments through HTTP.

**Stripe, Tempo, and MPP** focus on a payment method agnostic protocol for agents.

**Google and UCP** focus on commerce and a standard language for discovery through checkout. [Google's Universal Commerce Protocol](https://developers.google.com/pay/api/universal-commerce-protocol/overview){:target="_blank" rel="noopener noreferrer"} describes that broader journey.

**Google and AP2** focus on authorization and cryptographic evidence of what an agent was authorized to purchase. [Google's AP2 and UCP overview](https://codelabs.developers.google.com/next26/adk-agent-commerce){:target="_blank" rel="noopener noreferrer"} explains how the layers can work together.

**OpenAI and ACP** focus on agent commerce and connecting applications such as ChatGPT with merchant checkout. [OpenAI's product discovery announcement](https://openai.com/index/powering-product-discovery-in-chatgpt/){:target="_blank" rel="noopener noreferrer"} describes that direction, while [Stripe's ACP documentation](https://docs.stripe.com/agentic-commerce/protocol){:target="_blank" rel="noopener noreferrer"} covers the protocol.

**Visa and Mastercard** are extending existing payment networks and controls so agents can transact without throwing away the infrastructure that already moves trillions of dollars.

**PayPal** is extending merchant payments and commerce infrastructure to agents.

**Cloudflare** is working across Internet infrastructure, 402 payments, agent wallets, and web services.

**Circle** is focused on stablecoin infrastructure, programmable money, and settlement for machine transactions.

The important point is that these technologies are not necessarily substitutes for one another.

Google's UCP, for example, is designed to cover the broader commerce journey, while AP2 is a specialized payment and authorization layer that can work within UCP.

OpenAI's ACP is similarly focused on making merchant checkout accessible to applications such as ChatGPT.

Visa and Mastercard are approaching the problem from another direction: extending existing payment networks and controls so that agents can transact without throwing away the infrastructure that already moves money around the world.

This is why the emerging agentic payment stack is better understood as:

**Agent → Discovery → Identity → Authorization → Commerce → Payment Protocol → Payment Rail → Settlement**

HTTP 402 sits somewhere in the middle.

It does not replace the payment network.

It creates a standardized point where a service can say **payment is required**.

### The 402 directories

There is another piece beginning to appear around this ecosystem: directories.

If 402 makes services payable, agents still need to discover those services.

Emerging 402 directories are beginning to catalogue APIs and other services that accept protocols such as x402 and MPP. Some are machine readable, allowing an agent to search for a capability, understand its price, and call it directly.

That creates an interesting distinction.

A payment protocol answers:

**How do I pay?**

A directory answers:

**Where can I spend?**

Together, they begin to look like a discovery layer for a machine economy.

Instead of browsing an app store, an agent could search a directory for a capability, compare prices, pay for the one it wants, and use it immediately.

### The wallet problem

There is still one missing piece.

The agent needs money.

That is why programmable wallets are becoming part of the same conversation. A wallet gives an agent an economic identity and, more importantly, controlled authority to spend.

The model starts to look like:

**Human → Agent → Wallet → Service**

The human sets the rules. The agent gets a budget. A service asks for payment. The agent decides whether the transaction is worth making.

This is where wallets, payment protocols, and agent identity begin to converge.

The wallet answers **who can spend**.

The payment protocol answers **how they spend**.

The 402 response answers **what the service wants in return**.

### The bigger question

The competition may ultimately not be about whether x402 or MPP wins.

It may be about whether HTTP 402 becomes the common interface underneath all of them.

Traditional commerce starts with a checkout.

Machine commerce could start with an HTTP request.

The server says what it costs. The agent decides whether it is worth paying. A payment rail moves the money. The service returns the result.

That is a very different model of the Internet.

The web was built around:

**Request → Response**

The agentic web may increasingly look like:

**Request → Payment Required → Payment → Response**

A status code that sat mostly unused for decades may become part of the basic grammar of an Internet where machines are not just reading and calling services.

They are **buying from them**.

### References

1. Adnan Masood, [Agentic Payments 101: How the Card Payment System Works](https://medium.com/@adnanmasood/agentic-payments-101-1-2-how-the-card-payment-system-works-64b1a4eba9f2){:target="_blank" rel="noopener noreferrer"}
2. Adnan Masood, [Agentic Payments 101: Payment Standards and Protocols](https://medium.com/@adnanmasood/agentic-payments-101-2-2-payment-standards-and-protocols-acp-ucp-ap2-and-x402-26486e6d511f2){:target="_blank" rel="noopener noreferrer"}
3. [Stripe: Introducing the Machine Payments Protocol](https://stripe.com/blog/machine-payments-protocol){:target="_blank" rel="noopener noreferrer"}
4. [Google: Universal Commerce Protocol](https://developers.google.com/pay/api/universal-commerce-protocol/overview){:target="_blank" rel="noopener noreferrer"}
5. [Google: Secure Agent Commerce with AP2 and UCP](https://codelabs.developers.google.com/next26/adk-agent-commerce){:target="_blank" rel="noopener noreferrer"}
6. [OpenAI: Powering Product Discovery in ChatGPT](https://openai.com/index/powering-product-discovery-in-chatgpt/){:target="_blank" rel="noopener noreferrer"}
7. [Stripe: Agentic Commerce Protocol](https://docs.stripe.com/agentic-commerce/protocol){:target="_blank" rel="noopener noreferrer"}
8. [Cloudflare: Agent Payments](https://developers.cloudflare.com/agents/tools/payments/){:target="_blank" rel="noopener noreferrer"}
