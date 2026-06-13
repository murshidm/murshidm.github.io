---
layout: post
title: "Google’s Open Knowledge Format (OKF): Standardizing the LLM Wiki Pattern"
date: 2026-06-12
---

On June 12, 2026, Google announced the Open Knowledge Format (OKF), an open specification designed to make organizational knowledge portable and consumable by AI agents. In Google’s words, OKF formalizes the “LLM-wiki pattern” into a vendor-neutral format for representing metadata, business context, and curated knowledge that modern AI systems need. Google has also updated its Knowledge Catalog to ingest and serve OKF bundles to agents. 

[Introducing the Open Knowledge Format](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing?utm_source=chatgpt.com){:target="_blank" rel="noopener noreferrer"}

What caught my attention wasn’t the format itself, but the problem Google is acknowledging.

Over the last two years, enterprises have learned that the biggest limitation of AI isn’t model capability. It’s context.

Most organizations already possess the knowledge their AI systems need. It exists in documentation, business glossaries, data catalogs, architecture diagrams, dashboards, and the collective experience of employees. The challenge is that this knowledge is fragmented across systems and rarely represented in a way that AI can easily understand.

As companies deployed copilots and agents, a common pattern emerged. Teams started building dedicated knowledge repositories for LLMs — collections of business definitions, metric explanations, ownership information, data relationships, and organizational context. These repositories acted as a bridge between raw enterprise data and AI systems.

In many ways, every successful enterprise AI deployment ended up creating its own “LLM Wiki.”

The problem was that everyone implemented it differently. Some used Markdown. Others used YAML, vector databases, internal catalogs, or proprietary formats. Knowledge became locked inside tools and platforms.

Google’s answer is OKF.

The idea is surprisingly simple. If organizational knowledge is becoming a critical dependency for AI systems, it should be portable, interoperable, and standardized. Just as OpenAPI standardized how we describe APIs, OKF aims to standardize how we describe organizational knowledge. ([Google Cloud](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing?utm_source=chatgpt.com))

This becomes even more important as AI agents become more autonomous. An agent may have access to every tool and database in the company, but access alone doesn’t provide understanding. It still needs to know which metric is authoritative, which dataset is trusted, and how different concepts relate to each other.

That’s where OKF fits.

If MCP helps agents access systems, OKF helps agents understand them.

Whether OKF becomes widely adopted remains to be seen. But Google’s announcement signals something important: organizational knowledge is becoming a first-class asset in AI architectures. The next generation of enterprise AI won’t be limited by access to models. It will be limited by access to meaning.

And meaning is exactly what OKF is trying to standardize. ([Google Cloud](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing?utm_source=chatgpt.com))
