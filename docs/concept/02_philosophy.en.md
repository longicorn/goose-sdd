このドキュメントの[日本語版](./02_philosophy.md)もあります。

# Problem Organization and Solution Proposal

In [01_problem_context.md](/docs/concept/01_problem_context.md), we summarized the problems of existing development. Here, we will outline the solutions to those problems.

## Lack of Documentation

This is primarily caused by the high cost of document generation. As seen in the Vibe Coding example, AI-driven document generation can keep costs low. However, similar to Vibe Coding, a problem arises with the quality of the generated documents, which could be called "Vibe Documents." To avoid this, specifications must be written according to a certain format. This is the same as the past practice of writing documents correctly.
As long as the format is followed correctly, AI-driven document generation will likely be of high quality and low cost.
The idea of SDD development by AI is correct.

## Dependency Issues with Existing SDD Tools

The AI Agent I usually use is [Goose](https://github.com/block/goose).
Assuming the use of Goose helps solve the problems I will mention, so I'll state this first.

### Adopting Goose

Here are some features of Goose relevant to goose-sdd:

- As it says, "goose is your on-machine AI agent, capable of automating complex development tasks from start to finish," making it suitable for developers.
- It can integrate with numerous LLMs, and tools like Claude Code CLI, Gemini CLI, etc., are also available. This also helps solve API cost issues.
- Conversations with the AI are displayed in a simple, unadorned terminal screen, which is easy for a Vimmer like me to handle, unlike other CLIs.
- MCP and Agent SKILL are also available.
- The Recipe function allows for the automation of specific tasks.
  - It can be executed as a command with `goose run --recipe recipe.yaml`.
  - With `goose run --recipe recipe.yaml --interactive`, you can continue the conversation with the AI after the recipe has been executed.

For more details, please refer to the [Goose Document](https://block.github.io/goose/).

The ability to turn specific, routine processes into command-line commands eliminates the restriction of having to use other tools, unlike existing SDD tools.
Assuming this is possible, organizing recipes and command structures to simplify execution will enable the automation of various tasks.
`goose-sdd` is based on this idea and is considered optimal for realizing an SDD workflow using Goose.

## The Overly Complex Nature of Existing SDD Tools

I've seen the expression "using a sledgehammer to crack a nut" as feedback on existing SDD tools. There was a report where trying to fix a small bug (the nut) resulted in the tool generating a huge document (the sledgehammer) with "four user stories and a total of 16 acceptance criteria," which ended up creating more work. I share a similar impression.

I believe this is because they try to document the entire system, from each feature down to the minute details.
I think documentation should be properly separated and managed, just as humans have done in the past.
I have defined two layers: the overall system documentation (Macro) and the documentation for each feature or specific common architecture (Micro).

Following this, `goose-sdd` has broadly divided its commands into two: `goose-sdd --system ...` and `goose-sdd --feature ...`.
Furthermore, creating all documents at once can make the process slow and difficult to understand.

Therefore, taking `goose-sdd --system` as an example, we provide subcommands as follows, with one command generating one document. The `goose-sdd --feature` command follows the same principle.
Also, if your understanding evolves or the premises change, you can simply run the command again to update the corresponding document.
If you re-run a command to update a document, it is recommended to re-run the subsequent commands as well.

Incidentally, each command should take about a few tens of minutes for a small to medium-sized system, depending on its scale.
Furthermore, pre-generating the documents and having each command read them will make the conversation easier.

### init
Command: `goose-sdd --system init <language>`

Creates `docs/sdd/` and initializes each document.

### background
Command: `goose-sdd --system background`

Creates or updates `docs/sdd/00_product_background.md`.
It converses with the user to describe the system's background, purpose, overall picture, and concepts.

### concept
Command: `goose-sdd --system concept`

Creates or updates `docs/sdd/01_system_concept.md`.
It reads `docs/sdd/00_product_background.md` and describes the system's concept.
If more information is needed, it will converse with the user to deepen understanding before creating the document.

### architecture
Command: `goose-sdd --system architecture`

Reads these files and describes the system's architecture in `docs/sdd/02_system_architecture.md`.
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`

If more information is needed, it will converse with the user to deepen understanding before creating the document.

### rules
Command: `goose-sdd --system rules`

Reads these files and describes rules for collaboration with AI, coding conventions, testing policies, etc., in `docs/sdd/03_project_rules.md`.
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`
- `docs/sdd/02_system_architecture.md`

If more information is needed, it will converse with the user to deepen understanding before creating the document.

### glossary
Command: `goose-sdd --system glossary`

Reads these files and describes the domain glossary in `docs/sdd/04_domain_glossary.md`.
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`
- `docs/sdd/02_system_architecture.md`
- `docs/sdd/03_project_rules.md`

If more information is needed, it will converse with the user to deepen understanding before creating the document.

### The Drift Between Document and Code

This is the problem I mentioned in "Lack of Documentation." It also leads to a vicious cycle where developers, unable to understand the system, create code that "just works" without realizing there are issues. This makes understanding even more difficult, and subsequent code becomes even more problematic.
I also get the impression that such workplaces tend to avoid bearing the costs of product maintainability, such as refactoring and document generation.

To solve these problems, I believe it is crucial to make the document update approach (Flow) bidirectional.

- 1.  **Forward Sync (Design First / To-Be):**
    -   When adding new features or making modifications, the document is updated first, and the code is changed based on it.
    -   It functions as an instruction manual for the AI (Goose) and prevents implementation from going astray.
- 2.  **Reverse Sync (Reality First / As-Is):**
    -   A flow that "back-propagates" facts discovered during implementation or results from analyzing existing code into the documentation.
    -   When the code (reality) changes first, the document is immediately brought up to date with reality, resolving any drift.

goose-sdd maintains a state where "document and code are always consistent (Single Source of Truth)" by cycling through these "two flows."

In other words, `goose-sdd` should have the following principles:

*   **Single Source of Truth is Mutable:** The specification (Spec) is the authority, but if the code (Reality) changes first, the spec must be promptly updated to match the code (Reverse Sync).
*   **Verification:** Periodically check the consistency between the documents under `docs/sdd/` and the actual code. If any drift is found, recommend "updating the documentation."
