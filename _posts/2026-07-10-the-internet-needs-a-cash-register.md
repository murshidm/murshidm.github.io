---
layout: post
title: "The Internet Needs a Cash Register"
date: 2026-07-10
---

If agents become major users of the web, the Internet needs a way for them to pay.

Today, most web content is effectively binary: free or behind a subscription. That model works reasonably well for humans, but it is a poor fit for software that may consume hundreds of different resources to complete a task.

What if an article cost $0.001? A research query $0.05? A specialized dataset $1? A workflow $10?

This is where a machine native payment layer becomes interesting.

Cloudflare has been championing this direction. Its [Pay Per Crawl](https://blog.cloudflare.com/introducing-pay-per-crawl/) project is an early experiment in letting publishers charge AI crawlers for access rather than forcing them to simply allow or block them.

The experiment is important because it changes what an HTTP request can mean. A request does not have to be only **give me this resource**. It can also become **give me this resource, and here is what I am willing to pay for it**.

Imagine giving a research agent a $20 budget and asking it to find the best answer. It could spend a few cents on articles, fifty cents on a database query, a dollar on an API, and more on a specialized report.

It could make dozens of these decisions without asking a human to enter a credit card every time.

Humans don't behave this way because the transaction costs are too high.

Software can.

That is why the Internet needs more than authentication and authorization for the agentic era. It needs a **payment mechanism that works at the level of the request**.

The Internet already has protocols for moving information.

The next version needs a way to move **money alongside information**.

If the agentic web turns every resource into something a machine can discover and invoke, payment should become just as programmable.

The Internet needs a cash register.