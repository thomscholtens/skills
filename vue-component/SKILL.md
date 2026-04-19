---
name: vue-component
description: >
  Generate Vue 3 single-file components following project conventions: script setup with
  TypeScript, extracted Props interface, destructured props with defaults, scoped SCSS styles,
  typed emits. Use this skill whenever the user asks to create, scaffold, generate, or write
  a Vue component — even if phrased casually like "make me a button component", "write a card
  component", "I need a modal", or "can you create a XYZ component?". Also trigger when the user
  asks to add props, emits, or slots to an existing component. If Storybook is configured, also
  generate a co-located .stories.ts file for reusable components.
---
 
# Vue Component
 
## Component Anatomy
 
Every component follows this exact structure and order:
 
```vue
<script setup lang="ts">
// 1. Imports (external libs, then internal composables/utils)
 
// 2. Types & interfaces (Props + any other local types, grouped together)
interface MyComponentProps {
  // ...
}
 
// 3. defineProps + defineEmits
const { propA, propB = 'default' } = defineProps<MyComponentProps>()
 
// 3. Emits (only if needed)
const emit = defineEmits<{
  eventName: [argName: type]
}>()
 
// 4. Composables / reactive state
 
// 5. Computed properties
 
// 6. Methods / handlers
 
// 7. Lifecycle hooks (onMounted, etc.)
</script>
 
<template>
  <!-- single root element preferred, but not enforced -->
</template>
 
<style scoped lang="scss">
// scoped SCSS only — no global styles in component files
</style>
```
 
---
 
## Rules
 
### Props
 
- Always extract props to a named TypeScript interface: `<ComponentName>Props`
- Interface lives in the same `.vue` file (not in a separate types file) — position within the script block is flexible, e.g. it can be grouped with other interfaces/types at the top if there are several
- Use `defineProps<ComponentNameProps>()` — never `withDefaults`
- Set default values via destructuring:
```ts
// ✅ Correct
interface ButtonProps {
  label: string
  variant?: 'primary' | 'secondary' | 'ghost'
  disabled?: boolean
  size?: 'sm' | 'md' | 'lg'
}
const { label, variant = 'primary', disabled = false, size = 'md' } = defineProps<ButtonProps>()
```
 
```ts
// ❌ Never use withDefaults
const props = withDefaults(defineProps<ButtonProps>(), { variant: 'primary' })
```
 
- Required props have no `?` and no default
- Optional props always have `?` in the interface AND a default in destructuring
### Emits
 
Only define emits when the component actually emits events. Use typed interface syntax:
 
```ts
const emit = defineEmits<{
  change: [value: string]
  close: []
  select: [id: number, label: string]
}>()
```
 
### Slots
 
Document slots with a comment when non-obvious. Use `<slot>` in template directly — no need to define them in script unless you need to check `$slots` programmatically.
 
### Styles
 
- Always `<style scoped lang="scss">`
- No inline styles unless dynamically computed
- Use BEM-ish class naming for component internals: `.component-name__element--modifier`
- CSS custom properties for anything that might be themed
### Naming
 
- Component file: `PascalCase.vue` — e.g. `UserAvatar.vue`
- Props interface: `<ComponentName>Props` — e.g. `UserAvatarProps`
- Emits: verb-noun or verb — e.g. `update:modelValue`, `close`, `itemSelect`
- Internal refs/state: camelCase, descriptive — `isOpen`, `selectedIndex`
---
 
## Storybook Stories
 
### When to create a story
 
**Create a story** when the component is reusable — i.e. it could be used in more than one place, is part of a design system, or is a generic UI primitive (buttons, inputs, cards, modals, badges, etc.).
 
**Do NOT create a story** for:
- One-off page sections or layout components (`HeroSection.vue`, `CheckoutSummary.vue`)
- Page components (`pages/` directory)
- Components tightly coupled to a specific route or business context
If unsure, ask the user: *"Should I create a Storybook story for this? It looks like it could be reusable."*
 
### Story file conventions
 
- Filename: `MyComponent.stories.ts` — co-located next to `MyComponent.vue`
- Use CSF3 format with `satisfies Meta<typeof Component>`
- Always include a `Default` story
- Add more stories for meaningful variants (states, sizes, edge cases)
### Story template
 
```ts
import type { Meta, StoryObj } from '@storybook/vue3'
import MyComponent from './MyComponent.vue'
 
const meta = {
  title: 'Components/MyComponent',  // mirror directory structure in title
  component: MyComponent,
  tags: ['autodocs'],
  argTypes: {
    // only needed for args that need special controls (e.g. select, color)
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost'],
    },
  },
  args: {
    // sensible defaults shared across all stories
  },
} satisfies Meta<typeof MyComponent>
 
export default meta
type Story = StoryObj<typeof meta>
 
export const Default: Story = {
  args: {
    // minimal required props to render the default state
  },
}
 
export const Secondary: Story = {
  args: {
    variant: 'secondary',
  },
}
```
 
### `title` path convention
 
Mirror the component's directory path relative to `components/`:
- `components/base/Button.vue` → `title: 'Base/Button'`
- `components/form/TextInput.vue` → `title: 'Form/TextInput'`
- `components/Button.vue` → `title: 'Components/Button'`
---
 
## Full Example
 
**`BaseButton.vue`**
 
```vue
<script setup lang="ts">
interface BaseButtonProps {
  label: string
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  loading?: boolean
}
const {
  label,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
} = defineProps<BaseButtonProps>()
 
const emit = defineEmits<{
  click: [event: MouseEvent]
}>()
</script>
 
<template>
  <button
    class="base-button"
    :class="[`base-button--${variant}`, `base-button--${size}`, { 'base-button--loading': loading }]"
    :disabled="disabled || loading"
    @click="emit('click', $event)"
  >
    <span v-if="loading" class="base-button__spinner" aria-hidden="true" />
    <span class="base-button__label">{{ label }}</span>
  </button>
</template>
 
<style scoped lang="scss">
.base-button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
 
  &--primary { /* ... */ }
  &--secondary { /* ... */ }
  &--ghost { /* ... */ }
 
  &--sm { font-size: 0.75rem; }
  &--md { font-size: 0.875rem; }
  &--lg { font-size: 1rem; }
 
  &--loading { opacity: 0.7; pointer-events: none; }
 
  &__spinner { /* spinner styles */ }
  &__label { /* label styles */ }
}
</style>
```
 
**`BaseButton.stories.ts`**
 
```ts
import type { Meta, StoryObj } from '@storybook/vue3'
import BaseButton from './BaseButton.vue'
 
const meta = {
  title: 'Base/Button',
  component: BaseButton,
  tags: ['autodocs'],
  argTypes: {
    variant: { control: 'select', options: ['primary', 'secondary', 'ghost'] },
    size: { control: 'select', options: ['sm', 'md', 'lg'] },
  },
  args: {
    label: 'Button',
    variant: 'primary',
    size: 'md',
    disabled: false,
    loading: false,
  },
} satisfies Meta<typeof BaseButton>
 
export default meta
type Story = StoryObj<typeof meta>
 
export const Default: Story = {
  args: { label: 'Click me' },
}
 
export const Secondary: Story = {
  args: { variant: 'secondary', label: 'Secondary' },
}
 
export const Ghost: Story = {
  args: { variant: 'ghost', label: 'Ghost' },
}
 
export const Loading: Story = {
  args: { label: 'Saving...', loading: true },
}
 
export const Disabled: Story = {
  args: { label: 'Disabled', disabled: true },
}
```
 
---
 
## Checklist Before Outputting
 
- [ ] `<script setup lang="ts">` — no options API, no plain `<script>`
- [ ] Props interface named `<ComponentName>Props`
- [ ] `defineProps<ComponentNameProps>()` with destructuring for defaults
- [ ] No `withDefaults`
- [ ] Emits typed with interface syntax (only if emits exist)
- [ ] `<style scoped lang="scss">`
- [ ] Story file created if reusable (CSF3, co-located, same base name)
- [ ] Story `title` mirrors directory structure
