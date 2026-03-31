This document is also available in [Japanese](./02_philosophy.md).

# Philosophy: How `goose-sdd` Is Meant to Be Operated

In [01_problem_context.md](./01_problem_context.md), the need for Reverse SDD was framed.
This document explains the operating philosophy `goose-sdd` uses in response.

## 1. `goose-sdd` is an operational interface over Goose recipes

`goose-sdd` is not a large standalone platform.
It is a wrapper that organizes [Goose](https://github.com/block/goose) recipes
into a command structure that is easier to operate for SDD.

That gives it several advantages:

- it keeps Goose's interactive workflow
- model and provider selection stay aligned with Goose
- it can use well-known CLI-oriented providers such as Claude Code, Codex, and Gemini CLI, along with many other AI APIs
- it remains CLI-first instead of UI-bound
- behavior can be customized at the recipe level

So the essence of `goose-sdd` is not abstract SDD theory.
It is command-oriented operation that can survive real development work.

## 2. Specs are living documents, not static deliverables

`docs/sdd/` is not an archive of completed documents.
It is a working area updated jointly by humans and AI.

That means the workflow assumes:

- humans may edit the documents directly
- AI output may always be revised
- the same commands may be rerun to refresh documents
- temporary drift is tolerated, but not ignored

The goal is not to freeze correctness.
It is to keep the current understanding reflected.

## 3. The single source of truth is mutable

Even if `docs/sdd/` is treated as the single source of truth,
that does not mean it never changes.

Real operation includes both of these:

- humans define new intent and update the spec
- reality changes first in code, and the spec follows later

Because of that, `goose-sdd` keeps the principle that specs matter,
while also formally allowing specs to be updated when reality leads.

## 4. Information must be handled by confidence level

Once `goose-sdd --analyze` exists, the tool has to distinguish between types of information.

### Code-backed facts

Information derived from code, config, database schema, infrastructure definition,
or execution behavior. These are relatively verifiable.

### Mosaic information

Information from Notion, Confluence, meeting notes, conversation, memory, or requests.
It may be useful, but it is not truth by default.

### Approved understanding

Information that has been reviewed with a human and accepted as safe to use
for AS-IS understanding or future To-Be work.

The important point is that `mosaic` should not become spec automatically.
`goose-sdd` cares less about collecting fuzzy information than about promoting it safely.
The broader problem framing behind this is summarized in the related note
[The Information Mosaic](https://gist.github.com/longicorn/8f4d878eaecff4b0c4a1c964fc267056).

## 5. Commands are designed as small conversational units

`goose-sdd` follows a one-command, one-theme principle.
That is not only about convenience.

The purpose is to:

- keep each step small enough for human review
- make selective reruns possible
- keep AI context focused
- reduce failure modes caused by large one-shot generation

For example, the System Layer is split into background, concept, architecture, rules, and glossary.
The Feature Layer is split into requirements, design, test, code, and review.

## 6. `goose-sdd` has two main flows

### Forward Flow

This is the path from intent into implementation.

```text
background/concept/architecture
  -> requirement/design
  -> test/code/review
```

Here, `docs/sdd/` acts as the instruction set for AI-assisted implementation.

### Reverse Flow

This is the path from existing implementation back into explainable structure.

```text
discover -> feature
             |-> gather ---|
             |             |-> synthesize -> elevate
             |-> mosaic ---|
```

Here, `docs/sdd/analyze/` is the working area.
`gather` is the path for collecting facts from code and config,
while `mosaic` is a separate path for handling fragmented human-written information.
They can proceed independently, and `synthesize` is the stage that combines both.

## 7. The Analyze Layer is for negotiation readiness, not only reverse generation

The output of `--analyze` is AS-IS documentation.
But that is not the real endpoint.

Its deeper role is to:

- make current reality explainable
- help humans notice pain points and contradictions
- make it easier to decide where SDD should start in an existing codebase
- avoid writing To-Be documents on top of guessed As-Is assumptions

The important part is not to blur `gather` and `mosaic`.
`gather` expands code-backed facts.
`mosaic` handles fuzzy information safely.
Either path may advance first, but `synthesize` is explicitly designed to integrate both inputs.

That is why `elevate` matters.
It is the stage where teams decide what to adopt, what to revise, and what to discard.

## 8. Recommended operating model

### Greenfield work

- use `--system` to shape the whole system
- use `--feature` or `--implement` for delivery work
- if implementation runs ahead, use reverse-oriented updates to recover alignment

### Existing systems

- use `--analyze` to recover AS-IS
- let humans inspect drift and issues
- promote only the necessary parts into `--system` or `--feature`

### Ongoing operation

- not every code change needs a full documentation rewrite
- but drift should be corrected locally before it spreads
- treat SDD as ongoing understanding maintenance, not as an up-front ceremony

## 9. Summary

The philosophy of `goose-sdd` can be summarized like this:

- SDD should not be limited to greenfield work
- specs and code should synchronize in both directions
- information should be separated by confidence level
- documents should be maintained as living documents
- tools should stay small, rerunnable, and operable

With Reverse SDD in place, `goose-sdd` is no longer only a tool for creating specs first.
It becomes a tool for regaining and evolving specs while keeping development under control.
