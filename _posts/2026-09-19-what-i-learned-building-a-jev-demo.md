---
layout: post
title: "What I Learned Building a Jev Demo"
date: 2026-09-19
---

There has been a lot of excitement around decision-only models and what they might change about how we build AI systems.

But I think there is a useful distinction between a model being *capable* of something and a problem actually being a good fit for the model.

So I wanted to try a relatively boring problem:

> **Given a bank transaction and a set of receipts or invoices, which document matches the transaction?**

No chatbot. No generated UI. No image understanding. Just a bounded decision from a known set of options.

I put together a small demo to understand how [TypeSafe's](https://typesafe.ai/) Jev model works, how it compares with a traditional LLM, and where the system-design tradeoffs show up.

You can try the demo here:

[Jev transaction matching demo](https://jevdemo.dokket.app)

The original announcement is also worth reading for the motivation behind Jev:

[Original Jev announcement on X](https://x.com/CompleteSkeptic/status/2099925682726002904)

---

## First, what is Jev?

The easiest way I found to think about Jev is:

**A traditional LLM generates text. Jev answers structured questions.**

Instead of asking a model:

> "Look at these transactions and documents and tell me which ones match."

you define a question with a fixed set of possible answers.

TypeSafe describes Jev as a System One model designed for structured decisions: rather than sequentially generating text, it returns typed decisions with probabilities and confidence. Its current primitives are `choice`, `score`, and `noul`.

For my problem, `choice` is the natural fit.

I'm effectively asking:

> **Which one of these documents matches this transaction?**

That's a **pick one from a list** problem.

`score` is for rating something on a scale, while `noul` is for a yes/no-style probability. Neither describes the problem I'm trying to solve.

So the interesting part isn't simply that Jev can classify something. It's that you explicitly describe the **shape of the decision**.

[TypeSafe's documentation and primitives](https://typesafe.ai/)

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

## The `state`

The first important concept is **state**.

State is simply the shared background information that all the questions can use.

In this demo, the state is 1,515 characters containing the bank transactions and supporting documents:

```text
## Bank / card transactions

| # | Date       | Description                | Amount   |
|---|------------|----------------------------|----------|
| 1 | 2026-03-02 | RIO GRANDE POULTRY DIST    | -$1245.60 |
| 2 | 2026-03-04 | VAMONOS PEST CONTROL       | -$220.00  |
| 3 | 2026-03-06 | WWW.DUKECITYPOS.COM        | -$340.00  |
| 4 | 2026-03-09 | UBER TRIP 8823             | -$34.75   |
| 5 | 2026-03-11 | LAVANDERIA BRILLANTE       | -$318.90  |
| 6 | 2026-03-14 | SQ *CRYSTAL COFFEE CO      | -$22.50   |
| 7 | 2026-03-17 | DUKE CITY PRINT SHOP       | -$145.00  |
| 8 | 2026-03-19 | DOWNTOWN ABQ PARKING       | -$18.00   |
| 9 | 2026-03-22 | MADRIGAL ELECTROMOTIVE     | -$3250.00 |
|10 | 2026-03-25 | SUNLAND CLEANING SVCS      | -$180.00  |

## Supporting documents

- **doc_1** — Rio Grande Poultry & Produce
  - type: invoice
  - date: 2026-03-02
  - total: $1245.60

- **doc_2** — Vamonos Pest Control
  - type: invoice
  - date: 2026-03-04
  - total: $220.00

- **doc_3** — Duke City POS Systems
  - type: invoice
  - date: 2026-03-03
  - total: $340.00

- **doc_4** — Lavanderia Brillante
  - type: invoice
  - date: 2026-03-10
  - total: $318.90

- **doc_5** — Crystal Coffee Co
  - type: receipt
  - date: 2026-03-14
  - total: $22.50

- **doc_6** — Duke City Print Shop
  - type: invoice
  - date: 2026-03-16
  - total: $145.00

- **doc_7** — Madrigal Electromotive GmbH
  - type: invoice
  - date: 2026-03-20
  - total: $3250.00

- **doc_8** — Sunland Cleaning Services
  - type: invoice
  - date: 2026-03-25
  - total: $180.00

- **doc_9** — Desert Sun Signage Co
  - type: quotation/proforma
  - date: 2026-03-12
  - total: $375.00
```

Think of this as the **context for the entire job**.

It is sent once.

So if I have 10 transactions, I don't send the shared transaction/document information 10 separate times as `state`.

---

## Then come the questions

For every transaction, I create a `choice` question.

For example, Question 1 is:

```json
{
  "type": "choice",
  "instructions": "Transaction 1 on the statement is dated 2026-03-02, described as \"RIO GRANDE POULTRY DIST\", for USD 1245.60. Which of these documents is the supporting receipt or invoice for that exact transaction? Card descriptors are often abbreviated or carry a payment-processor prefix, and an invoice may be dated a few days before it settles.",
  "criteria": {
    "doc_1": "invoice from Rio Grande Poultry & Produce, dated 2026-03-02, USD 1245.60",
    "doc_2": "invoice from Vamonos Pest Control, dated 2026-03-04, USD 220.00",
    "doc_3": "invoice from Duke City POS Systems, dated 2026-03-03, USD 340.00",
    "doc_4": "invoice from Lavanderia Brillante, dated 2026-03-10, USD 318.90",
    "doc_5": "receipt from Crystal Coffee Co, dated 2026-03-14, USD 22.50",
    "doc_6": "invoice from Duke City Print Shop, dated 2026-03-16, USD 145.00",
    "doc_7": "invoice from Madrigal Electromotive GmbH, dated 2026-03-20, USD 3250.00",
    "doc_8": "invoice from Sunland Cleaning Services, dated 2026-03-25, USD 180.00",
    "doc_9": "quotation/proforma from Desert Sun Signage Co, dated 2026-03-12, USD 375.00",
    "none": "No document in the list is a receipt or invoice for this transaction. Choose this when the closest document is only superficially similar, is a quotation or estimate that was never paid, or covers different goods, a different vendor, or a materially different amount."
  }
}
```

So the question is essentially:

> Transaction 1 is this transaction. Here are the possible documents. Which one matches?

Then there is Question 2, Question 3, and so on.

With 10 transactions, I have **10 questions**.

Importantly, these aren't 10 separate API calls.

They are sent together in **one Jev request** and evaluated in parallel.

Conceptually:

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

That's one of the things I wanted to understand with the demo.

"One call" doesn't mean there is only one question.

It means **many structured decisions can be sent together and processed in parallel**. TypeSafe describes this parallel sampling as a key part of the System One approach. 

---

# What do you get back?

For each `choice` question, Jev returns more than just the selected document.

You get:

- the selected option
- a confidence value
- the probability assigned to each possible option

For example, the raw response for the first transaction looks like this:

```json
{
  "type": "choice",
  "choice": "doc_1",
  "confidence": 1,
  "probabilities": {
    "doc_3": 0,
    "doc_1": 1,
    "none": 0,
    "doc_5": 0,
    "doc_9": 0,
    "doc_4": 0,
    "doc_6": 0,
    "doc_2": 0,
    "doc_8": 0,
    "doc_7": 0
  }
}
```

So instead of just getting:

> `doc_1`

you get something closer to:

> `doc_1` — confidence 1.00

plus the probability distribution over the choices.

That's useful because you can see not only **what Jev chose**, but **how the decision was distributed across the available choices**.

In this particular example the decision is extremely clear: `doc_1` gets probability 1 and every other option gets 0.

The demo also exposes the raw JSON response so you can see what actually came back rather than only seeing a cleaned-up result.

---

# The first interesting result

On my original run, I had:

- **10 transactions**
- **9 documents**
- **10/10 correct matches**

The two systems agreed on every transaction.

Jev took: **1.01 seconds**

The traditional LLM took: **4.12 seconds**

So Jev was about: **4.1× faster**

The measured cost was: **Jev: $0.000275** versus: **LLM: $0.003552**

which is about: **12.9× cheaper**

Both systems got the same **10/10** result against the sample answer key.

The detailed run looked like this:

| | Jev | Traditional LLM |
|---|---:|---:|
| Decision time | 1.01s | 4.12s |
| Input tokens | 6,551 | 886 |
| Output tokens | 1,059 | 178 |
| Estimated cost | $0.000275 | $0.003552 |
| Correct | 10/10 | 10/10 |

These are measurements from these two runs on the same files and through the same network path. They are **not a published benchmark**.

---

# But then I looked at the tokens

This is where things became more interesting.

Jev used: **6,551 input tokens** and: **1,059 output tokens**

The LLM used: **886 input tokens** and: **178 output tokens**

So Jev actually moved substantially **more tokens**.

Approximately: **7.4× more input tokens**
and: **6× more output tokens**

At first glance, that seems strange.

If Jev is faster and cheaper, why is it processing more tokens?

The answer is in how the request is structured.

---

# Why is Jev's input larger?

In the current demo, each question contains the list of documents it can choose from.

So with 10 transactions, the document information gets repeated across the 10 questions.

The same document can therefore appear in the shared state **and** inside multiple question definitions.

For example, Question 1 contains:

```json
"criteria": {
  "doc_1": "invoice from Rio Grande Poultry & Produce, dated 2026-03-02, USD 1245.60",
  "doc_2": "invoice from Vamonos Pest Control, dated 2026-03-04, USD 220.00",
  "doc_3": "invoice from Duke City POS Systems, dated 2026-03-03, USD 340.00",
  "...": "..."
}
```

But that same document information already exists in the shared `state`.

That's redundant.

So this isn't a fundamental requirement of the problem.

It's an inefficiency in my current implementation.

---

# Why is Jev's output larger?

This one is different.

A `choice` response includes the probability distribution over the available choices.

With 10 choices, that means roughly 10 probability values for each question.

With 10 questions:

**10 × 10 = 100 probability values**

The raw response actually contains those distributions:

```json
"probabilities": {
  "doc_3": 0,
  "doc_1": 1,
  "none": 0,
  "doc_5": 0,
  "doc_9": 0,
  "doc_4": 0,
  "doc_6": 0,
  "doc_2": 0,
  "doc_8": 0,
  "doc_7": 0
}
```

The traditional LLM in my test was simply asked to return the matching document and a confidence number.

So its output was much smaller.

This explains a lot of the **1,059 vs. 178 output-token** difference.

The extra output isn't necessarily verbosity. It is structured information.

TypeSafe's documentation describes `choice` as returning a distribution over the choices along with confidence. 

---

# Then I did a pricing sanity check

This is where I think it's important not to mix up **speed** and **cost**.

Jev's measured usage was:

**6,551 input + 1,059 output tokens**

If I pretend those tokens were charged using the LLM's pricing of:

- $2 / million input tokens
- $10 / million output tokens

then Jev's actual token usage would cost approximately: **$0.0237**

The traditional LLM run cost: **$0.003552**

So under those rates, Jev would actually be about: **6.6× more expensive.**

That's a useful result.

It shows that **Jev's speed and Jev's cost are two different things**.

The speed comes from the way the model processes structured decisions: non-autoregressively and in parallel.

The low cost in my original test comes from Jev's current pricing model: input is priced at **$0.042 per million tokens**, while output is currently free. TypeSafe publishes those rates in its announcement and pricing materials. 

So I wouldn't describe the demo as proving:

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

**50 × 30 = 1,500 possible matches**

You don't send 1,500 API requests.

You still send **one Jev request**.

But that request contains:

**50 questions**

with roughly:

**30 choices per question**

So there are around **1,500 transaction/document combinations** inside that request.

This distinction matters.

The API call count stays at one, but the amount of decision work grows with the number of transactions and choices.

---

# What might the token usage look like?

Using my 10-transaction/9-document run as a rough baseline, scaling the problem up to 50 transactions and 30 documents could put the request somewhere around:

**~100,000 input tokens**

and:

**~16,000 output tokens**

Those are **not measured results**. They are an extrapolation intended to show the shape of the scaling.

The actual numbers would depend on the size of the state, question instructions, document descriptions, and response structure.

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
    Jev
     |
Final decision
```

The important system-design question becomes:

> **How many choices should Jev actually have to consider for each decision?**

That's a much more useful question than simply asking how many rows are in the bank statement.

---

# There's also a hard limit

There is another practical constraint: TypeSafe currently documents a **64k-token limit** for the state plus all questions in a call.

That means a sufficiently large statement will eventually outgrow the single-call approach.

You would then need to split the work across multiple calls.

So there are two different scaling concerns:

**Cost/scaling**

> More transactions × more choices = more work.

**Request size**

> Eventually the combined state and questions reach the per-call limit.

Shorter questions and smaller choice sets give you more headroom, but they don't remove the underlying scaling problem.

---

# The pattern TypeSafe recommends

Interestingly, the solution to the choice-set problem isn't some trick I invented.

TypeSafe's workflow examples use the same general idea: decompose a larger task into smaller decisions and use code for deterministic work where possible. Their published workflow guidance explicitly describes retrieving the relevant state, asking narrow questions, and using the results programmatically. 

That leads to a natural architecture:

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

For example, application code could determine that only **2–4 documents** are plausible for a particular transaction based on simple things like date, amount, and currency.

Jev then answers:

> "Which of these 2–4 candidates is the right one?"

rather than:

> "Search through 30 documents and figure it out."

That's a cleaner division of responsibilities.

And it changes the scaling question.

Instead of every transaction having 30 possible documents, each transaction might have only 2–4 candidates.

The model's choice set stays small even as the overall document collection grows.

---

# One thing I'd change in my own demo

The current implementation repeats document descriptions inside each question.

That's something I'd fix before treating the benchmark numbers as final.

The better design is to keep the detailed document information in the shared `state` and make the question's choices as small and focused as possible.

For example, the current question:

```text
doc_1 → full document description
doc_2 → full document description
doc_3 → full document description
...
```

could instead use terse choice identifiers while relying on the shared state for the descriptions.

The key idea is:

```text
STATE
  └── full document descriptions

QUESTION
  ├── doc_1
  ├── doc_2
  ├── doc_3
  └── none
```

That should reduce the redundant input.

It won't eliminate the output-token difference, though, because the probability distribution is part of the `choice` response.

I'd also rerun the comparison after making this change rather than assuming how much it will improve the numbers.

---

# What I think the demo actually taught me

The most interesting thing I got from this exercise isn't:

> **"Jev is 12.9× cheaper."**

That statement is too dependent on the current pricing and this particular implementation.

And it's not simply:

> **"Jev is 4.1× faster."**

That's a measurement from one workload.

The more useful lesson for me is about **problem shape**.

Jev is interesting when you can describe your problem as:

> **Here is the context. Here is a bounded set of choices. Pick one.**

And when designing around it, the size of that choice set matters.

My original demo had:

**10 transactions × 9 documents**

which is manageable.

At:

**50 transactions × 30 documents**

you have:

**1,500 possible matches**

inside one request.

That doesn't automatically make Jev unsuitable. But it does mean **choice-set size becomes a system-design concern**.

And that is probably the biggest thing I learned from actually putting one of these systems together rather than just reading about it.

---

# Final takeaway

The cleanest mental model I have now is:

**Traditional LLM:**

> Give it context → ask it to reason → generate an answer.

**Jev:**

> Give it context → define the possible answers → let it choose.

That difference has consequences for **latency, token usage, pricing, scaling, and architecture**.

And those consequences are much easier to understand when you can see the actual request and response rather than just looking at a headline like "4× faster" or "13× cheaper."

That's ultimately what I wanted the demo to show.

[Try the Jev transaction matching demo](https://jevdemo.dokket.app)