---
layout: post
title: "What I Learned Building a Jev Demo"
date: 2026-09-19
---

There has been a lot of excitement around decision-only models and what they might change about how we build AI systems.

But I think there is a useful distinction to make between a model being *capable* of something and a problem actually being a good fit for the model.

So I wanted to try a relatively boring problem:

> **Given a bank transaction and a set of receipts or invoices, which document matches the transaction?**

No chatbot. No generated UI. No image understanding. Just a bounded decision from a known set of options.

I put together a small demo to understand how [TypeSafe](https://typesafe.ai/)'s Jev model actually works, how it compares with a traditional LLM, and where the system-design tradeoffs show up.

You can try the demo here:

[Jev transaction matching demo](https://jevdemo.dokket.app)

The original Jev announcement is also worth reading for the motivation behind the model:

[Original Jev announcement on X](https://x.com/CompleteSkeptic/status/2099925682726002904)

---

## First, what is Jev?

The easiest way I found to think about Jev is:

**A traditional LLM generates text. Jev answers structured questions.**

Instead of asking a model:

> "Look at these transactions and documents and tell me which ones match."

you define a question with a fixed set of possible answers.

TypeSafe describes its System One models as non-autoregressive models that return typed answers with probability scores rather than generated text. The current documentation describes three primitives: `choice`, `score`, and `noul`.

[TypeSafe documentation](https://typesafe.ai/)

For my problem, `choice` is the natural fit.

I'm effectively asking:

> **Which one of these documents matches this transaction?**

That's a **pick-one-from-a-list** problem.

`score` is designed for rating something along a spectrum, while `noul` is for a yes/no-style probability. Neither really describes the problem I'm trying to solve.

So the interesting part isn't just that Jev can classify something. It's that you can explicitly describe the **shape of the decision**.

---

# How the demo works

The architecture is deliberately simple.

I have:

1. A bank statement
2. A collection of receipts/invoices
3. Text extracted from those files
4. A shared `state`
5. One `choice` question per transaction
6. One Jev request containing all the questions

The document extraction happens before the timing measurement. I'm using Kreuzberg WASM to extract the text and then normalizing it into markdown.

Jev doesn't see the original PDF or image. It sees the resulting text.

### The `state`

The first important concept is **state**.

State is simply the shared background information that all the questions can use.

In this demo it contains things like:

```text
[PLACEHOLDER: INSERT ACTUAL `state` MARKDOWN HERE]

Instruction:
Show the exact state passed to Jev, including the transactions
and document descriptions.

Add a short comment explaining what each section represents.
```

Think of this as the **context for the entire job**.

It is sent once.

So if I have 10 transactions, I don't send the transaction/document information 10 separate times as shared state.

---

## Then come the questions

For every transaction, I create a `choice` question.

For example:

```text
[PLACEHOLDER: INSERT ACTUAL QUESTION 1 JSON HERE]

Instruction:
Show the real JSON payload for Question 1 from the demo.

Explain in one or two sentences:
- what transaction is being matched
- what the available choices are
- what Jev is being asked to decide
```

Then there is Question 2, Question 3, and so on.

With 10 transactions, I have **10 questions**.

Importantly, these aren't 10 separate API calls.

They are sent together in **one Jev request** and evaluated in parallel.

So conceptually:

```text
              ONE JEV REQUEST

                    STATE
                      |
          +-----------+-----------+
          |                       |
      Question 1             Question 2
      Question 3             Question 4
          ...                   ...
      Question 9             Question 10

                    |

              10 answers
```

That's one of the things I was trying to understand with the demo.

"One call" doesn't mean there is only one question.

It means **many structured decisions can be sent together and processed in parallel**.

---

# What do you get back?

For each `choice` question, Jev returns more than just the selected document.

You get:

- the selected option
- a confidence value
- the probability assigned to each possible option

For example, conceptually:

```text
[PLACEHOLDER: INSERT ACTUAL RAW RESPONSE FOR ONE QUESTION]

Instruction:
Show a shortened version of one real response, preserving the
chosen option, confidence, and probability distribution.

Explain what each field means in plain English.
```

So instead of just getting:

> `document_17`

you can get something closer to:

> `document_17` — confidence 0.96

along with the probabilities for the other choices.

That is useful because you can see not only **what Jev chose**, but **how the model's decision was distributed across the available choices**.

The raw JSON response in the demo is there specifically so you can see what actually came back rather than only seeing a cleaned-up result.

---

# The first interesting result

On my original run, I had:

- **10 transactions**
- **9 documents**
- **10/10 correct matches**

The Jev run took:

**1.01 seconds**

The traditional LLM run took:

**4.12 seconds**

So Jev was about:

**4.1x faster**

The measured cost was:

**Jev: $0.000275**

versus:

**LLM: $0.003552**

which is about:

**12.9x cheaper**

Both systems got the same **10/10** result on this dataset.

These are measurements from this particular run, not a general benchmark. That's an important distinction.

---

# But then I looked at the tokens

This is where things became more interesting.

Jev used:

**6,551 input tokens**

and:

**1,059 output tokens**

The LLM used:

**886 input tokens**

and:

**178 output tokens**

So Jev actually moved substantially **more tokens**.

Approximately:

**7.4x more input tokens**

and:

**6x more output tokens**

At first glance, that seems strange.

If Jev is faster and cheaper, why is it processing more tokens?

The answer is in how the request is structured.

---

# Why is Jev's input larger?

In the current demo, each question contains the list of documents it can choose from.

So if I have 10 transactions, the document information gets repeated across the 10 questions.

The same document may therefore appear in the shared state **and** inside multiple question definitions.

For example:

```text
[PLACEHOLDER: INSERT ACTUAL QUESTION 1 `criteria` OBJECT]

Instruction:
Highlight how the document descriptions appear in the criteria.
```

Then compare that with:

```text
[PLACEHOLDER: INSERT ACTUAL STATE SECTION]

Instruction:
Highlight where the same document information already exists in state.
```

That's redundant.

The model already has the document information in the shared state, but I'm repeating parts of it in each question.

So this isn't a fundamental requirement of the problem.

It's an inefficiency in my current implementation.

---

# Why is Jev's output larger?

This one is different.

Jev's `choice` response includes the probability distribution over the available choices.

So if there are 10 possible documents, the response can contain roughly 10 probability values for that question.

With 10 questions, that's potentially around:

**10 x 10 = 100 probability values**

The traditional LLM in my test was simply asked to return the matching document and a confidence number.

So its output was much smaller.

This explains a lot of the **1,059 vs 178 output-token** difference.

The extra output isn't necessarily verbosity. It's structured information.

---

# Then I did a pricing sanity check

This is where I think it's important not to mix up **speed** and **cost**.

Jev's measured token usage was:

**6,551 input + 1,059 output**

If I pretend those tokens were charged using the LLM's pricing of:

- $2 / million input tokens
- $10 / million output tokens

then Jev's actual token usage would cost approximately:

**$0.0237**

The traditional LLM run cost:

**$0.003552**

So under those rates, Jev would actually be about:

**6.6x more expensive.**

That's a useful result.

It shows that **Jev's speed and Jev's cost are two different things**.

The speed comes from the way the model processes the structured decisions: non-autoregressively and in parallel.

The low cost in my original test comes from Jev's current pricing model: input is priced at **$0.042 per million tokens**, while output is currently free.

So I wouldn't describe the demo as proving that:

> "Jev does less work than an LLM."

The numbers don't support that.

A better description is:

> **Jev processes this kind of structured decision differently, which makes it fast, while its current pricing makes that execution inexpensive for this workload.**

Those are separate claims.

---

# What happens when the problem gets bigger?

This is where the system-design question gets interesting.

Imagine:

**50 bank transactions**

and:

**30 receipts/invoices**

At first, you might think:

> "That's just 50 line items."

But that's not the whole picture.

Each transaction is being compared against the available documents.

So you effectively have:

**50 x 30 = 1,500 possible matches**

You don't send 1,500 API requests.

You still send **one Jev request**.

But that request contains:

**50 questions**

with roughly:

**30 choices per question**

So there are around **1,500 transaction/document combinations** for Jev to evaluate.

---

# What might the token usage look like?

Using my 10-transaction/9-document run as a very rough baseline, scaling the problem up to 50 transactions and 30 documents could put the request somewhere around:

**~100,000 input tokens**

and:

**~16,000 output tokens**

Those are **not measured results**. They're an extrapolation to show the shape of the scaling.

The actual number would depend on the size of the state, question instructions, document descriptions, and response structure.

The important point isn't whether the estimate is exactly 100,000.

It's this:

> **One API call does not mean constant cost.**

The amount of work inside that call still grows as the number of questions and choices grows.

---

# This led me to a bigger Jev design principle

If you're designing a system around Jev, **keep the choice set small**.

A bounded decision is where the model starts to make architectural sense.

Instead of thinking:

> "I have 30 documents, so every transaction should have 30 choices."

you can think about the system as:

```text
Transaction
     |
Possible choices
     |
Jev makes the decision
```

The important design question becomes:

> **How many choices should Jev actually have to consider for each decision?**

That is a much more useful question than simply asking how many rows are in the bank statement.

---

# There's also a hard limit

There is another practical constraint: TypeSafe currently documents a **64k-token limit** for the state plus all questions in a call.

That means a sufficiently large statement will eventually outgrow the single-call approach.

You would then need to split the work across multiple calls.

So there are two different scaling concerns:

**Cost/scaling:**

> More transactions x more choices = more work.

**Request size:**

> Eventually the combined state and questions reach the 64k-token limit.

Shorter questions and smaller choice sets give you more headroom, but they don't remove the underlying limit.

---

# The pattern TypeSafe recommends

Interestingly, the solution to the choice-set problem isn't some trick I invented.

TypeSafe's own guidance is essentially:

> **retrieve and filter in code first, then send the model only what the question needs.**

[TypeSafe's documentation and primitives](https://typesafe.ai/)

That leads to a very natural architecture:

```text
Bank transactions + documents
              |
       Application logic
              |
       Small candidate set
              |
             Jev
              |
       Final decision
```

For example, application code might determine that only **2-4 documents** are plausible for a particular transaction based on simple things like date, amount, and currency.

Jev then answers:

> "Which of these 2-4 candidates is the right one?"

rather than:

> "Search through 30 documents and figure it out."

That's a much cleaner division of responsibilities.

---

# One thing I'd change in my own demo

The current implementation repeats document descriptions inside each question.

That's something I'd fix before treating the benchmark numbers as final.

The better design is to keep the detailed document information in the shared `state` and make the question's choices as small and focused as possible.

For example:

```text
[PLACEHOLDER: INSERT REVISED `criteria` CODE HERE]

Instruction:
Show the revised version where document IDs are the choices and
their full descriptions remain in state.
```

This should reduce the redundant input.

It won't eliminate the output-token difference, though, because the probability distribution is part of the `choice` response.

I'd also rerun the benchmark after this change rather than assuming how much it will improve the numbers.

---

# What I think the demo actually taught me

The most interesting thing I got from this exercise isn't:

> **"Jev is 12.9x cheaper."**

That statement is too dependent on the current pricing and this particular implementation.

And it's not simply:

> **"Jev is 4.1x faster."**

That's a measurement from one workload.

The more useful lesson for me is about **problem shape**.

Jev is interesting when you can describe your problem as:

> **Here is the context. Here is a bounded set of choices. Pick one.**

And when designing around it, the size of that choice set matters.

My original demo had:

**10 transactions x 9 documents**

which is manageable.

At:

**50 transactions x 30 documents**

you have:

**1,500 possible matches**

inside one request.

That doesn't automatically make Jev unsuitable. But it does mean **choice-set size becomes a system-design concern**.

And that is probably the biggest thing I learned from actually putting one of these systems together rather than just reading about it.

---

## Final takeaway

The cleanest mental model I have now is:

**Traditional LLM:**

> Give it context -> ask it to reason -> generate an answer.

**Jev:**

> Give it context -> define the possible answers -> let it choose.

That difference has consequences for **latency, token usage, pricing, scaling, and architecture**.

And those consequences are much easier to understand when you can see the actual request and response rather than just looking at a headline like "4x faster" or "13x cheaper."

That's ultimately what I wanted the demo to show.

[Try the demo](https://jevdemo.dokket.app)
