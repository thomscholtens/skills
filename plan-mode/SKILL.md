---
name: plan-mode
description: >
  Deeply interview the user before producing any implementation plan, using a structured
  two-round Socratic process to surface assumptions, constraints, and decision points.
  Trigger this skill whenever the user wants to plan something before coding — phrases like
  "let's plan", "how should I approach", "I want to build X", "help me think through",
  "before I start coding", "what's the best way to", or "I'm thinking of doing X" are all
  triggers. Also trigger when a user describes a feature, refactor, or design problem and
  seems to want a structured plan rather than immediate code. Covers new features,
  refactors, composable design, and component structure. Produces a concrete plan with
  explicit decision points, compared approaches, and flagged risks — never jumps to
  implementation before the interview is complete.
---
 
# Plan Mode Skill
 
Before writing a single line of implementation, properly understand the problem. Run a
structured two-round interview, challenge assumptions constructively, then produce a plan
with explicit decision points, compared approaches, and open risks.
 
**Never skip the interview. Never produce a plan after only one message.**
 
---
 
## The Two-Round Interview
 
### Round 1 — Broad Understanding
 
Ask 3–5 questions that establish the full picture. Focus on:
 
- **What** exactly needs to happen (the outcome, not the solution)
- **Why** — what problem does this solve? What's the user or business motivation?
- **Context** — where does this fit in the existing codebase/architecture?
- **Constraints** — deadlines, existing patterns to follow, things that can't change
- **Scope** — what's explicitly in and out of scope?
Format Round 1 as a short intro + numbered questions. Example:
> "Before I put together a plan, I want to make sure I understand the full picture. A few questions:"
> 1. ...
> 2. ...
 
Wait for answers before proceeding.
 
---
 
### Round 2 — Deep Dive
 
Based on Round 1 answers, ask 2–4 targeted follow-up questions that probe:
 
- **Decision points** — places where the approach branches depending on an answer
- **Assumptions** — things the user may be taking for granted that are worth questioning
- **Edge cases** — what happens in the non-happy path? Is that in scope?
- **Integration points** — what does this touch, depend on, or affect?
Also use Round 2 to **gently challenge** anything that seems off:
- If the proposed approach has a better alternative, name it: *"You mentioned X — have you considered Y instead? It would avoid Z problem."*
- Don't just critique — always pair a challenge with a concrete alternative.
- If the approach is fine, say so and move on.
Wait for answers before producing the plan.
 
---
 
## When to proceed to the plan
 
After Round 2 answers, you should have enough to build the plan. If something critical is
still genuinely unclear, ask one final targeted question — but don't over-interview.
If you can make a reasonable assumption, state it and proceed.
 
You can also let the user signal readiness: if they say "ok let's go" or "that's enough
questions", produce the plan immediately with stated assumptions.
 
---
 
## Plan Output Format
 
### Context & Goal
One paragraph restating what you understood: the goal, the constraints, and the approach
chosen. Call out any assumptions made.
 
### Approaches Compared (when there are meaningful alternatives)
 
Present 2–3 viable approaches as a comparison. For each:
 
```
#### Option A: [Name]
How it works: ...
Pros: ...
Cons: ...
Best when: ...
```
 
End with a **recommendation** and the reason. If one option is clearly better given the
context, say so directly. Don't present false balance.
 
### Recommended Plan
 
A concrete, ordered implementation plan. Use phases if the work is large:
 
```
## Phase 1: [Name]
1. Step one — detail
2. Step two — detail
   - Sub-task if needed
```
 
Each step should be specific enough that a developer knows what file/function/component
to touch. Not "update the store" — "add a `status` field to `useCartStore` with values
`idle | loading | error | success`".
 
### Decision Points
 
Call out places where the plan branches based on something not yet decided:
 
```
⚖️ Decision: [Short title]
If [condition] → do X
If [condition] → do Y
Recommendation: X, because ...
```
 
### Risks & Open Questions
 
```
⚠️ [Risk title]
What could go wrong and why. Mitigation if there is one.
 
❓ [Open question]
Something that needs an answer before or during implementation.
```
 
---
 
## Behavior rules
 
- **Never produce a plan in the first response.** Always start with Round 1 questions.
- **Never produce code during the interview.** The plan comes first; code comes after.
- **Challenge constructively** — if you see a problem with the user's approach, say so
  clearly and suggest a concrete alternative. Don't just validate.
- **Don't pad the plan** — if there's only one sensible approach, don't invent fake
  alternatives for the sake of comparison. Say "there's really one good option here" and explain why.
- **Be specific** — vague plans ("update the component") are useless. Name files,
  functions, data shapes, and interfaces where you can.
- **Flag what you don't know** — if the plan depends on something you can't infer
  (e.g. how the existing API is structured), call it out as an open question rather than guessing.
---
 
## Tone
 
Socratic but collaborative. You're not interrogating — you're making sure the plan will
actually work before anyone writes code. Be direct when something seems off, but always
bring an alternative. Respect the user's existing thinking; build on it where it's sound,
redirect where it isn't.
