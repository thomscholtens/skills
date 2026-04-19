---
name: progress-tracker
description: >
  Create and maintain a living progress file (PROGRESS.md or FEATURE-X.md) while an AI
  agent builds something. Trigger this skill whenever the user starts a non-trivial task
  and wants to track what's being built — phrases like "track your progress", "keep a log",
  "document what you're doing", "make a progress file", "I want to be able to pick this up
  later", or "write down what you did" are all triggers. Also trigger proactively when a
  task has multiple steps, touches multiple files, or will likely span more than one session.
  The file serves both as a human-readable build log and as a machine-readable session
  handoff — the next Claude session should be able to read it and continue the work.
---
 
# Progress Tracker Skill
 
Maintain a living progress document throughout an agentic task. Update it as work happens —
not just at the end. The file must be useful to both a human reviewing what was done and to
a future Claude session picking up where this one left off.
 
---
 
## File naming & placement
 
| Task size / type | File name | Location |
|---|---|---|
| Single focused feature | `FEATURE-[name].md` | Next to the feature, or project root |
| Large multi-part task | `PROGRESS.md` | Project root |
| Quick exploratory task | `PROGRESS.md` | Project root |
 
When in doubt: use `PROGRESS.md` at project root. If the user specifies a name or location,
use that.
 
---
 
## When to create the file
 
Create the progress file **before doing any work** — as the very first action. A blank or
minimal file is fine to start. This ensures the file exists even if the session is cut short.
 
---
 
## When to update the file
 
Update after:
- Each meaningful step completed (a component built, a function written, a file refactored)
- Any unexpected problem encountered or dead end hit
- A significant decision made (especially when alternatives were considered)
- A phase or milestone reached
- The session ends or is about to end
Do **not** wait until everything is done. Incremental updates are the whole point.
 
---
 
## File structure
 
Use this structure. Sections marked `(required)` must always be present.
Sections marked `(as needed)` only appear when relevant.
 
```markdown
# [Task / Feature Name] (required)
 
> **Status:** [Not started | In progress | Blocked | Complete]  
> **Last updated:** [timestamp or session note]  
> **Started:** [timestamp or session note]
 
## Goal (required)
What this task is trying to achieve. One short paragraph — specific enough that someone
unfamiliar with the conversation can understand the objective.
 
## Approach (required, written before work starts, updated if it changes)
The chosen strategy and why. If alternatives were considered and rejected, note that here
briefly.
 
## Progress (required)
 
### ✅ Done
- [Step or milestone completed] — brief note on what was built/changed and why
 
### 🔄 In Progress
- [What's currently being worked on]
 
### 📋 To Do
- [What still needs to happen, in order if relevant]
 
## Files Changed (as needed)
List files created, modified, or deleted — with a one-line note on what changed.
- `path/to/file.ts` — created: new composable for X
- `path/to/component.vue` — modified: added error state handling
 
## Decisions Made (as needed)
Record non-obvious choices made during the task.
 
### [Decision title]
**Chose:** [what was done]  
**Because:** [reason]  
**Alternatives considered:** [what else was possible and why it was rejected]
 
## Dead Ends & Problems (as needed)
Honest record of things that didn't work, bugs hit, or approaches abandoned.
 
### [Problem title]
**What happened:** ...  
**Why it failed / was abandoned:** ...  
**What was done instead:** ...
 
## Blockers (as needed)
Things that are preventing progress and need external input.
- [Blocker description] — needs: [what's required to unblock]
 
## Handoff Notes (required when session ends or task is incomplete)
Everything a fresh Claude session needs to pick this up:
- Current state of the work
- The next concrete action to take
- Any context that isn't obvious from the code
- Gotchas or things to watch out for
- Files to read first before continuing
```
 
---
 
## Handoff note quality
 
The handoff section is the most important part for multi-session work. A good handoff note
answers: *"If I open this file cold with no conversation history, what do I need to know to
continue this task without asking the user to re-explain everything?"*
 
A good handoff includes:
- The exact next step (not "continue the refactor" — "refactor `useCartStore.ts`, starting
  at the `checkout()` function which currently has no error handling")
- Any in-progress state (e.g. "the component renders but the emit is not yet wired up")
- Relevant file paths to read first
- Any decisions that haven't been finalized yet
- Known issues or constraints discovered during the session
---
 
## Tone & style
 
- Write for a reader who has **zero conversation context** — don't assume they saw the chat
- Be specific: file names, function names, component names — not "updated the store"
- Dead ends and wrong turns are valuable — don't omit them
- Keep entries concise but complete — bullet points over prose for the log sections
- The Goal and Approach sections can be prose; everything else should be scannable
---
 
## Example: minimal in-progress file
 
```markdown
# Cart Checkout Refactor
 
> **Status:** In progress  
> **Last updated:** Session 1  
> **Started:** Session 1
 
## Goal
Refactor the checkout flow to separate UI state from business logic. Currently
`CheckoutForm.vue` directly calls the API and manages all state internally.
Target: pure UI component + `useCheckout` composable handling all logic.
 
## Approach
Extract logic into `composables/useCheckout.ts`. Pass loading/error/success state
down as props. Wire up emits for submit. Delete direct API call from component.
 
## Progress
 
### ✅ Done
- Created `useCheckout.ts` with `submit()`, `status` ref, and error handling
 
### 🔄 In Progress
- Updating `CheckoutForm.vue` to use the composable instead of inline logic
 
### 📋 To Do
- Remove direct `useFetch` call from `CheckoutForm.vue`
- Add empty state for when cart is empty
- Test error state renders correctly
 
## Files Changed
- `composables/useCheckout.ts` — created
- `components/CheckoutForm.vue` — in progress
 
## Decisions Made
 
### Composable vs Pinia store
**Chose:** Local composable  
**Because:** Checkout state doesn't need to persist across routes  
**Alternatives considered:** Pinia store — overkill for ephemeral form state
 
## Handoff Notes
Next action: open `CheckoutForm.vue` and replace the `useFetch` call (line ~34) with
`const { submit, status, error } = useCheckout()`. The composable is complete and tested.
Watch out: the form currently uses `v-model` on a prop — that needs to become an emit.
```
