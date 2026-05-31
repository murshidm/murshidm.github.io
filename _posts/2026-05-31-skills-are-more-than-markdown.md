---
layout: post
title: "Skills Are More Than Markdown: Seeding Small Business Data for Agents"
date: 2026-05-31
categories: [AI, Agents, Developer Tools]
---   

For my app, I needed a constant stream of daily business data: receipts, invoices, spreadsheets, reports, and text logs simulating a small business operating day in and day out. I didn’t need perfect, polished documents. What I needed was the ability to consume contextually consistent business records on the fly for testing.

That turned out to be surprisingly difficult. Small business data is notoriously hard to find. Real company records are private, while public datasets on Kaggle and Hugging Face are usually fragmented and disconnected. I even searched Google for random business datasets, but found little that could recreate a realistic business operation. Transactions come from one source, product catalogs from another, and invoice samples from a third. Stitching them into a coherent operational history for a single business requires significant manual effort.

At this point, generating the data myself no longer made sense. The whole reason I was building with AI was to automate this kind of tedious work. I wanted a coding agent to create the business records, maintain the datasets, and continuously expand the simulated company (i.e. a bakery) as needed.

But there was a catch.

If you simply ask an agent to “generate some bakery data,” you get hallucinations. Dates mismatch, products change names, tax rates fluctuate randomly, and customer names on invoices don’t match the customer database. The output looks realistic at first glance, but it lacks the consistency and context that make a business feel real.

To solve this, I built a custom agent skill called bakeryseed.

Many think of agent skills as simple markdown files a `SKILL.md` containing a system prompt or a list of instructions. But a robust, production grade skill can be a complete software package. 

By utilizing the global setup directory at `.agents/skills/` (specifically `~/.agents/skills/bakeryseed`), I created a self contained skill that pairs agent instructions with Node scripts, templates, and automated validation. 

Now, when I need to seed my development environment, I simply pull up my coding agent and run the slash command: `/bakeryseed`.

### Keeping the Agent Grounded: Context and Knowledge

The secret to preventing hallucinations and ensuring consistency is a clear boundary of context. 

The `bakeryseed` skill relies on two core concepts:
1. **AGENTS.md**: A file copied to the project root during initialization. It contains the specific business profile the bakery's name, address, tax rate, employee list, and product catalog. The agent and scripts read this file to ground every generated document in the same reality.
2. **A Living Knowledge Base**: A `knowledge/` directory that stores structural templates and is updated constantly, allowing the generation logic to evolve alongside the business rules.

Because the generation script reads the profile from `AGENTS.md` and coordinates with the agent's behavior, the output stays highly consistent. The bread pricing on a Tuesday invoice matches the catalog price, and the supervisor signing off on a PDF report is a real employee listed in the system profile.

### Anatomy of a Multi File Agent Skill

To make this work, I packaged the logic into a proper agent skill rather than a giant prompt. The skill includes instructions, generation code, validation scripts, templates, and test cases. Together, these components give the agent a structured workflow for creating consistent business records instead of inventing everything from scratch.

```text
bakeryseed/
├── README.md                  # Quick start guide and usage instructions
├── SKILL.md                   # Skill definition, invocation rules, workflow
├── package.json               # Dependencies and npm scripts
│
├── scripts/
│   ├── bakeryseed.mjs         # Main generation engine
│   ├── install-deps.sh        # Dependency installer
│   └── verify-output.mjs      # Output validation checks
│
├── templates/
│   └── AGENTS.template.md     # Customizable bakery profile template
│
├── references/                # Static reference assets and examples
│
└── evals/
    └── evals.json             # Test scenarios and expected outputs
```

The key idea is that the agent does not start from a blank slate. It reads the business profile, follows the workflow defined in SKILL.md, generates the required documents through the execution engine, and then validates the results before returning them. That structure is what keeps customer names, products, dates, inventory levels, and financial records consistent across days of generated data.

### How the Pieces Fit Together

The operational flow of the skill is designed to be seamless:

1. **Installation**: You clone or copy the `bakeryseed` directory into your global agent skills folder (`~/.agents/skills/bakeryseed`) and run `npm install` (using the convenience script `install deps.sh`).
2. **Initialization**: Running `/bakeryseed init` invokes `bakeryseed.mjs` to establish the directory structure (`output/`, `knowledge/`) and copies `templates/AGENTS.template.md` to your root workspace as `AGENTS.md`. You can then customize the name of your bakery, the address, and the products.
3. **Data Generation**: When you run `/bakeryseed` for a specific date, the script reads `AGENTS.md`, generates a full day of operations, and outputs a suite of varied document formats (receipts, CSV sales logs, Word invoices, PDF summaries, ZIP backups) under `output/DD MM YYYY/`.
4. **Verification**: The validation script `verify output.mjs` runs post generation to check that all required formats were written and conform to standard schemas.
5. **Evaluation**: The `evals/` directory enables automated tests against different dates and configurations to guarantee the skill continues to behave properly as the generator scripts are updated.

### Moving Beyond Prompting

By wrapping prompting (`SKILL.md`), state management (`AGENTS.md`), and programmatic file generation (Node scripts) into a unified skill package, we bridge the gap between AI capability and structured software. The agent acts as the coordinator and contextual glue, while the underlying scripts do the heavy lifting of raw document creation.

This pattern isn't limited to bakery data. You can build multi file skills for seeding medical records, financial logs, or repository test suites. 

I’m planning to upload the complete `bakeryseed` skill repository publicly. In a follow up article, I will walk through the code of the generation scripts and detail exactly how to write your own custom, multi file agent skills.