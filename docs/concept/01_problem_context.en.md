This document is also available in [Japanese](./01_problem_context.md).

# Context: Why `goose-sdd` Needs Both Forward and Reverse Flows

This document explains the problem setting behind `goose-sdd`.
The short version is simple: modern development cannot be handled with `Spec -> Code` alone.
It also needs `Code -> Spec` so teams can understand existing systems and update specs to match reality.

## Problem 1: Code grows, but understanding does not

In many teams, the codebase grows every day.
What does not accumulate at the same pace is the explanation around it:

- why the design was chosen
- which constraints shaped it
- what is intentional and what is compromise
- where the actual feature boundaries are

In that state, a codebase may exist, but it is not an explainable system.
New contributors are forced to read code first and spend a long time building context.

## Problem 2: Documents exist, but lose trust quickly

The issue is not only missing documentation.
More often, the problem is partial and unreliable documentation:

- old design docs exist, but they no longer match the code
- there is information in tickets, Notion, or chat logs, but truth and noise are mixed
- important decisions are scattered across PR comments or oral history

So the problem is not just absence.
It is the lack of information organized into a trustworthy form.

## Problem 3: AI increases change velocity, but also understanding debt

AI coding tools accelerate implementation.
They also amplify a different kind of risk:

- vague prompts still produce working code
- implementation moves faster than intent is recorded
- change volume can outpace human review capacity
- nobody can clearly explain why the system looks the way it does

This is more than technical debt.
It is better described as understanding debt.

## Problem 4: Traditional SDD is strong for greenfield work, weak for adoption midstream

Many SDD approaches assume `Spec -> Code`.
That assumption is useful, but incomplete for real-world development.

Existing products usually have conditions like these:

- a large codebase already exists
- specs are missing or outdated
- discoveries during implementation change the design
- writing To-Be docs without understanding As-Is leads to paper architecture

That means teams need SDD that can start from reality.

## Problem 5: If the workflow is too heavy, it will not survive operation

One reason SDD-style workflows are resisted is operational weight:

- they ask for too many documents at once
- even small changes require large rituals
- fully delegating the process to AI turns it into a black box

When that happens, the workflow is introduced but not sustained.

## The assumptions `goose-sdd` makes

`goose-sdd` is built on the following assumptions.

### 1. Specs are authoritative, but not immutable

Specs matter.
But if code reveals the current reality first, the specs must be updated too.
The single source of truth is not a stone tablet. It is an actively maintained artifact.

### 2. Code-backed facts and human-supplied fuzzy information must be handled separately

Facts derived from code, config, and execution are not the same as
facts derived from memory, conversation, or external documents.

If they are mixed carelessly, AS-IS documents also become unreliable.
That is why `goose-sdd` treats analysis results and mosaic information separately.

### 3. Both Forward and Reverse SDD must be first-class flows

- Forward SDD: move from specs and design into code
- Reverse SDD: recover the current spec from the codebase

Only with both flows can the tool support both greenfield and existing development.

### 4. The workflow must be split into human-reviewable units

Instead of generating everything at once, `goose-sdd` breaks the process into
background, concept, architecture, requirements, design, and other smaller units.
This is not only for AI. It is mainly to keep each step understandable and reviewable by humans.

## Reverse SDD in this context

`goose-sdd --analyze` is not just a code summarizer.
It has three jobs:

- recover the current AS-IS view from existing code
- sort fragmented knowledge into trustworthy facts versus noisy claims
- connect those findings back into future Forward SDD work

So Reverse SDD is the entry point for bringing an existing system into an SDD workflow.

## Summary

`goose-sdd` is not a tool for producing more documents.
It is an operational model for closing the understanding gap between code and documentation
so humans can keep the system under control.

That requires both of these:

- Forward SDD for design-led work
- Reverse SDD for reality-led recovery

The next document explains how `goose-sdd` turns that problem setting into an actual philosophy and command structure.
