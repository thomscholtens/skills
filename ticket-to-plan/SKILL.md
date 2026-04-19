---
name: ticket-to-plan
description: >
  Create a structured implementation plan markdown file from an Azure DevOps ticket description.
  Investigates the problem, explores the codebase to find relevant files, and produces a concrete
  plan with sections for Problem, Investigation, Files to Change, Implementation Steps, and
  Validation. Trigger this skill whenever the user pastes an Azure DevOps ticket, work item,
  bug report, or feature description and wants a plan — phrases like "create a plan for this
  ticket", "plan this work item", "here's the ticket", "turn this into a plan", or any time
  the user pastes what looks like a ticket description (acceptance criteria, repro steps, user
  story format). Also trigger when the user says "plan this bug" or "plan this feature".
---
 
# Ticket to Plan
 
Turn an Azure DevOps ticket into a structured, actionable implementation plan saved as a
markdown file. The plan should be specific enough that a developer can pick it up cold and
know exactly what to read, change, and verify.
 
**Never produce the plan immediately.** Always investigate first.
 
---
 
## Phase 1 — Parse the Ticket
 
Read the ticket carefully and extract:
 
- **Type**: bug fix, new feature, refactor, chore, or unclear
- **Core problem or goal**: what is broken or what needs to exist
- **Acceptance criteria**: what does "done" look like
- **Affected area**: any hints about which part of the codebase is involved (component names, page names, API routes, feature names mentioned in the ticket)
- **Ambiguities**: anything underspecified, contradictory, or that requires a decision
If the ticket type or core goal is genuinely unclear after reading, ask one focused
clarifying question before proceeding. Otherwise, state your interpretation and move on.
 
---
 
## Phase 2 — Investigate the Codebase
 
Use available tools (filesystem, search, code reading) to explore the codebase and answer:
 
### For bug tickets:
1. **Reproduce the path** — trace the code path described in the ticket. Find the component,
   composable, store, or server route where the bug likely lives.
2. **Find the root cause** — read the relevant code. What is actually wrong? Is it a wrong
   assumption, a missing guard, a race condition, a wrong data shape?
3. **Check for related bugs** — are there other places in the codebase with the same pattern
   that might have the same issue?
### For feature tickets:
1. **Find the existing foundation** — what already exists that this feature builds on?
   Components, composables, API routes, types, stores.
2. **Find the integration points** — what does this new code need to connect to?
3. **Find similar existing features** — is there a precedent in the codebase to follow?
### For both:
- Note the specific file paths, component names, function names, and line ranges you find
- Note what you expected vs what you found if there's a mismatch
- Note any patterns or conventions in the affected area you should follow
**Be thorough but targeted.** Don't summarise the whole codebase — focus on what's directly
relevant to the ticket.
 
---
 
## Phase 3 — Produce the Plan File
 
### File naming & placement
 
| Ticket type | File name | Location |
|---|---|---|
| Bug fix | `PLAN-fix-[short-description].md` | Project root |
| New feature | `PLAN-feat-[short-description].md` | Project root |
| Refactor / chore | `PLAN-refactor-[short-description].md` | Project root |
| Unclear | `PLAN-[short-description].md` | Project root |
 
`short-description` should be kebab-case, 2–4 words, derived from the ticket title.
Example: `PLAN-fix-cart-total-rounding.md`, `PLAN-feat-user-avatar-upload.md`
 
If the user specifies a different location or name, use that instead.
 
---
 
### Plan file structure
 
```markdown
# Plan: [Ticket Title]
 
> **Ticket**: [ID if available] — [Title]
> **Type**: Bug fix | Feature | Refactor | Chore
> **Created**: [date]
 
---
 
## Problem
 
[1–3 sentences. What is wrong or what needs to exist. Written from a user/product perspective,
not a code perspective. Restate the ticket's intent in your own words to confirm understanding.]
 
---
 
## Investigation
 
[What you found when exploring the codebase. Be specific — file paths, function names,
component names, line numbers where helpful. This is your working notes made readable.]
 
### Root cause / current state
[For bugs: what is actually broken and why.
For features: what exists today and what's missing.]
 
### Relevant files found
[A list of files directly relevant to this work, with a one-line note on why each matters.]
 
- `path/to/file.ts` — [why it matters]
- `path/to/Component.vue` — [why it matters]
 
### Patterns & conventions to follow
[Any conventions in the affected area worth noting — naming, data shapes, how similar
things are done — so the implementation stays consistent.]
 
---
 
## Implementation Steps
 
[Ordered steps. Each step names the exact file(s) to touch and what to do. Specific enough
that a developer knows what to open. Not "update the composable" — "add a `roundToNearest`
helper in `app/utils/currency.ts` and call it in `useCart.ts` line 42".]
 
### Phase 1: [Name if multiple phases, omit if single phase]
 
- [ ] Step one — `path/to/file.ts`: what to do and why
- [ ] Step two — `path/to/Component.vue`: what to change
  - Sub-detail if needed
 
### Phase 2: [Name] *(if needed)*
 
- [ ] ...
 
---
 
## Edge Cases & Risks
 
[Things that could go wrong, non-happy paths to handle, or decisions not yet made.
Use these markers:]
 
⚠️ **[Risk title]** — What could go wrong. Mitigation if known.
 
⚖️ **[Decision needed]** — What needs to be decided. Options and recommendation.
 
❓ **[Open question]** — Something that needs an answer before or during implementation.
 
*(Omit this section if there are genuinely no risks or open questions.)*
 
---
 
## Validation
 
[How to verify the work is correct and complete. Should map to the ticket's acceptance
criteria. Include:
- Manual steps to test the happy path
- Edge cases to test manually
- Any automated tests to write or update
- What "done" looks like]
 
### Manual testing
- [ ] [Step to verify the main scenario]
- [ ] [Edge case to test]
 
### Automated tests
- [ ] [Test to write or update, with file path if known]
 
*(If no automated tests are needed or applicable, note that explicitly.)*
```
 
---
 
## Writing rules
 
- **Be specific** — file paths, function names, component names. Vague plans are useless.
- **Investigation comes before opinion** — don't guess at root causes. Read the code first.
- **Flag what you don't know** — if you couldn't find something relevant, say so in the
  Investigation section as an open question rather than silently guessing.
- **Steps should be executable** — each step should be clear enough that the developer knows
  which file to open and what to do when they get there.
- **Validation must be concrete** — "test that it works" is not validation. Name the scenario,
  the input, and the expected output.
- **Keep it tight** — don't pad sections. If a section has nothing meaningful to say, keep it
  to one sentence or omit it (except Problem, Investigation, Steps, and Validation which are always required).
