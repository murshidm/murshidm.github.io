---
layout: post
title: "Why Wiki-style Knowledge is Better Than RAG for Agents"
date: 2026-05-08
categories: [AI, Agents, Knowledge Management]
---

I’ve been thinking about this a lot lately, and I keep ending up in the same place: for most agent workflows, the wiki-style knowledge structure just feels more natural and durable than classic RAG.

RAG had its moment and it still works but in practice it often feels like a temporary architectural compromise. You chunk documents, embed them, retrieve top-k, and hope the answer is somewhere in that semantic neighborhood. It’s clever, but it’s also a bit fragile. Context gets fragmented, relevance is probabilistic, and the system never really “understands” how knowledge connects.

The wiki pattern feels different. Instead of treating knowledge as scattered embeddings to be fetched on demand, you structure it like a living system of interconnected pages more like how humans actually maintain shared understanding. Agents don’t just retrieve snippets; they navigate a knowledge graph that can evolve, accumulate structure, and support reasoning over time.

What’s interesting is that the industry is slowly drifting in this direction. Even tools like Pinecone’s Nexus are pointing toward more agent-native knowledge systems rather than pure vector retrieval. You can see the shift away from “search for context” toward “maintain and reason over a structured memory.”

There’s also a growing discourse around this in the agent community. Andrej Karpathy has been vocal about the idea that LLMs paired with persistent, structured knowledge bases almost wiki-like in nature are a more scalable direction than naive RAG pipelines. Not as a rejection of retrieval, but as an upgrade in how knowledge is organized and maintained over time.

Karpathy method:
- [Andrej Karpathy: LLM wiki knowledge base discussion](https://www.mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code)

To me, the key difference is this: RAG is about finding knowledge, while wiki-style systems are about accumulating and structuring it. And for agents that are supposed to operate over long time horizons, the latter starts to matter a lot more.

RAG wasn’t wrong it just feels increasingly like an intermediate step.
