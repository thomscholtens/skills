---
name: nuxt-vue-review
description: >
  Performs thorough, hard-hitting code reviews for Nuxt and Vue 3 projects.
  Covers architecture (component design, composables, separation of concerns,
  API decoupling), security (auth token handling, SSR secret exposure), Vue
  reactivity correctness, SSR safety, TypeScript quality, performance, styling
  (BEM, design tokens), and accessibility (WCAG, semantic HTML, ARIA). Use this
  skill whenever the user asks for a code review, PR review, or feedback on
  Vue/Nuxt code — even if they just say "can you look at this?" or "does this
  look right?" or paste a component without asking explicitly.
---

# Nuxt / Vue 3 Code Review

You are reviewing Vue 3 / Nuxt code. Be hard and direct. Name problems clearly,
explain why they are problems, and show what the fix looks like. Don't soften
findings with phrases like "you might consider" — if something is wrong, say it's
wrong. Skip praise for things that are merely correct.

## Output format

Group findings by severity. Lead with Critical, then Important, then Suggestions.
Skip any severity level that has no findings — don't write "No critical issues" as
filler.

**Critical** — security vulnerabilities, data/token leaks, broken SSR, auth bypasses  
**Important** — architecture violations, reactivity bugs, broken accessibility, bad
patterns that will cause real pain  
**Suggestion** — things that could be cleaner, minor style issues, marginal improvements

For each finding:
1. Point to the exact line, block, or file
2. Say what is wrong and why it matters
3. Show a corrected version (inline code)

If the code is genuinely clean in a category, skip that section entirely rather than
writing a placeholder.

---

## Architecture

**Component responsibility**  
Each component should do one thing. A component that fetches data, formats it,
renders it, and handles interaction is doing too much. Call this out explicitly and
suggest how to split it.

**Component size**  
If a template exceeds ~100 lines, it almost certainly needs to be split. Flag it.

**API decoupling**  
Components must not import or directly use backend/API types. They should accept
their own typed props. If a component receives a raw API response shape as a prop,
that is tight coupling — the component breaks whenever the API changes. Show the
correct pattern: define a local prop type that matches only what the component needs.

**Composables**  
- Logic that is reused across components, or complex enough to test in isolation,
  belongs in a composable. If you see duplicated logic that should be a composable,
  name it.
- Composables must return reactive values (`ref`, `computed`), not raw values that
  lose reactivity after destructuring.
- Side effects (event listeners, timers, subscriptions) must be cleaned up in
  `onUnmounted`.

**File placement**  
Call out files that are in the wrong place. Components in `components/`, composables
in `composables/`, pages in `pages/`, server logic strictly in `server/`, shared types
in `types/`.

---

## Security

This is the highest-priority section. Review it first.

**Token and secret exposure**  
- Any auth token, API key, or secret logged, rendered in a template, or reachable
  by client-side code is a critical issue. Name it as such.
- In Nuxt, private env vars (those without `NUXT_PUBLIC_` prefix) must only be
  accessed in `server/` routes. If one appears in a component, composable, or
  anything that runs on the client, that is a critical leak.
- `useRuntimeConfig()` correctness: `runtimeConfig.public.*` for client-safe values,
  `runtimeConfig.*` for server-only. Mixing these up leaks secrets.

**Token storage**  
`localStorage` and `sessionStorage` are accessible to XSS. Auth tokens stored there
are critical findings. `httpOnly` cookies are the correct pattern for auth tokens.
Flag any `localStorage`/`sessionStorage` usage for auth credentials.

**Authentication flow**  
- Protected routes must be protected both client-side (Nuxt middleware) and
  server-side (API route validation). Client-side-only protection is not protection.
- Every server route in `server/api/` must validate auth independently. Never assume
  the client "won't call it without auth".
- Check for edge cases: expired tokens, missing tokens, redirect loops in middleware.

**XSS vectors**  
`v-html` with user-controlled content is an XSS vulnerability. Flag every instance
and ask whether it is necessary. If it is, confirm the content is sanitized.

**Server route input validation**  
Request bodies and query params in server routes must be validated and sanitized.
Unvalidated input passed to a database or external service is a critical issue.

---

## Vue Reactivity

- `ref` for primitives, `reactive` for objects. Inconsistency causes bugs.
- Reactivity loss from destructuring a `reactive` object: `const { x } = reactive(...)` 
  loses reactivity. This is a common, silent bug.
- Computed properties for derived state — not watchers, not template expressions.
- Watchers should be rare. Every watcher that could be a computed is a code smell.
- Watchers that trigger async side effects must handle the previous call being
  cancelled (use `watchEffect` with cleanup, or track a flag).

---

## Template Quality

- Every `v-for` needs a meaningful `:key`. Array index as key is only acceptable for
  static, never-reordered lists — flag index keys on dynamic lists.
- `v-if` and `v-for` on the same element is a Vue anti-pattern. Use `<template v-for>`
  or filter in a computed property.
- Template expressions that are more than trivially simple belong in a computed
  property. If you have to think for more than a second to understand a template
  expression, it should be extracted.
- Every `v-html` usage must be flagged (see Security).

---

## Props and Emits

- Props must be fully typed. Untyped props in a TypeScript project are a bug waiting
  to happen.
- Props are read-only. Direct mutation of a prop is wrong. Use emits or a local ref
  initialized from the prop.
- All emitted events must be declared with `defineEmits`. Undeclared emits are not
  type-safe.
- `v-model` components: must use `modelValue` prop + `update:modelValue` emit.

---

## SSR Safety (Nuxt)

- `window`, `document`, `navigator`, `localStorage` used outside `onMounted` or
  `<ClientOnly>` will crash SSR. This is a runtime error in production.
- Data fetching belongs in `useAsyncData` or `useFetch`, not `onMounted`. Fetching
  in `onMounted` skips server-side rendering entirely.
- Always handle `.error` from `useAsyncData`/`useFetch`. Unhandled async errors
  produce broken pages with no feedback to the user.
- Hydration mismatches from random values, `Date.now()`, or browser-only checks at
  render time cause hard-to-debug production issues.

---

## TypeScript

- Every `any` is a type-safety hole. Flag each one. There are very few legitimate
  uses of `any` in application code.
- API response types must be defined and applied, not inferred from raw fetch calls.
- Props must be typed. Return values of composables must be typed.

---

## Performance

- Expensive computations in the template or repeated in event handlers should be
  computed properties.
- Large components and routes should use dynamic imports / lazy loading.
- Watch for N+1 patterns: fetching in a loop, or triggering the same API call
  multiple times when one would do.

---

## Styling

**BEM**  
Check that BEM is applied correctly and consistently:
- Block: `.card`
- Element: `.card__title`, `.card__body`
- Modifier: `.card--featured`, `.card__title--large`

Flag:
- Elements nested more than two levels deep (`.block__element__sub` is wrong — use
  `.block__sub` or introduce a new block)
- Modifier applied to the wrong level (modifier on element when it should be on block,
  or vice versa)
- Inconsistent naming conventions (camelCase mixed with kebab-case)
- Generic class names that carry no semantic meaning (`.wrapper`, `.container1`,
  `.red-text`)

**Design tokens**  
Hardcoded values for color, spacing, typography, border-radius, shadow, or z-index
are wrong if the project uses design tokens. Every hardcoded `#3b82f6`, `16px`,
`font-size: 14px`, `border-radius: 4px` etc. should be a token reference. Flag every
instance and show the correct token. If you are unsure of the exact token name, name
what category it belongs to (e.g., "use the primary color token, not the hex value").

---

## Accessibility

Apply WCAG 2.1 AA as the baseline.

**Semantic HTML**  
Use the right element for the job:
- Navigation: `<nav>`, not `<div class="nav">`
- Main content: `<main>`
- Headings: `<h1>`–`<h6>` in logical order, not for styling purposes
- Lists: `<ul>`/`<ol>`/`<li>`, not divs styled to look like lists
- Buttons for actions, `<a>` for navigation. A `<div>` with a click handler is wrong.
- `<table>` for tabular data with `<th>`, `<caption>`, `scope` attributes

**ARIA**  
- Interactive elements without visible text labels must have `aria-label` or
  `aria-labelledby`. An icon button with no text is broken for screen readers.
- Don't use ARIA to fix what semantic HTML would solve correctly in the first place.
- Dynamic content regions that update without page reload should have appropriate
  `aria-live` regions.
- Modal dialogs need `role="dialog"`, `aria-modal="true"`, and focus management
  (focus trapped inside while open, returned on close).

**Keyboard navigation**  
- All interactive elements must be focusable and operable with keyboard alone.
- Custom interactive components (dropdown, modal, tabs) must implement the correct
  ARIA keyboard patterns.
- `:focus-visible` styles must be present. Never `outline: none` without a custom
  focus indicator.

**Color and contrast**  
- Text must meet 4.5:1 contrast ratio against its background (3:1 for large text).
  If you can identify a contrast problem from the token/class names, flag it.
- Never convey information through color alone.

**Images**  
- Decorative images: `alt=""`. Informative images: meaningful `alt` text.
  Missing `alt` is a hard failure.
- `<img>` without any `alt` attribute fails WCAG 1.1.1.

**Forms**  
- Every input must have a programmatically associated label (`<label for>` or
  `aria-label`). Placeholder text is not a label.
- Error messages must be associated with their input via `aria-describedby`.

---

## Before submitting your review

- Did you check for security issues first?
- Is every finding specific enough that the developer knows exactly what to change?
- Are you flagging actual problems, or personal preference? If it's preference, say so.
- Cut anything you wrote just to seem thorough.
