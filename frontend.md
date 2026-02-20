# The Frontend Doctrine

> Principles governing all frontend architecture decisions across all projects.

---

## The Supreme Law

### ALL REAL LOGIC BELONGS IN THE BACKEND

This is non-negotiable. Violation is **HOCHVERRAT** (high treason).

```
BACKEND owns:                    FRONTEND owns:
- Business logic                 - UI components
- Calculations                   - User interactions
- Validations                    - Display formatting
- AI/LLM calls                   - Ephemeral UI state
- Data transformations           - Calling backend endpoints
- Decision making                - Rendering results
- Database operations            - NOTHING ELSE
```

**The frontend is a THIN INTEGRATION LAYER.** It calls backend endpoints and displays results. That's it.

**If you're writing logic in the frontend, STOP and ask:**
- Should this calculation be an endpoint?
- Should this validation be server-side?
- Am I duplicating backend logic?

**The only "logic" allowed in frontend:**
- Pure display formatting (date formatting, number formatting)
- UI state management (modals open/closed, which tab is active)
- Calling the right endpoint with the right parameters

**Examples of HOCHVERRAT:**
```javascript
// VERBOTEN - Price calculation in frontend
const totalPrice = items.reduce((sum, i) => sum + i.price * i.qty, 0);

// VERBOTEN - Validation logic in frontend
if (email.match(/regex/) && password.length >= 8) { ... }

// VERBOTEN - Business rules in frontend
const canEdit = user.role === 'admin' || user.id === item.ownerId;

// VERBOTEN - AI prompt building in frontend
const prompt = `Generate description for ${listing.address}...`;
```

**Correct approach:**
```javascript
// GUT - Backend calculates, frontend displays
const { totalPrice } = await api.getCartTotal(cartId);

// GUT - Backend validates, frontend shows errors
const { valid, errors } = await api.validateForm(data);

// GUT - Backend decides, frontend respects
const { canEdit } = await api.getPermissions(itemId);

// GUT - Backend does AI, frontend shows result
const { description } = await api.generateDescription(listingId);
```

---

### DUPLICATION IS DIE HÖCHSTE VERRAT (The Highest Treason)

**No frontend development proceeds without thorough component structure planning.**

Before writing ANY new component, you MUST answer:
1. **Does this already exist?** Search the codebase. Search hard.
2. **Should this be a component at all?** Or is it just inline markup?
3. **Can an existing component be extended?** Props, slots, variants?
4. **Is this a pattern we want to establish?** Will it be reused?

**Before ANY refactoring:**
- Will this introduce duplication? If yes, STOP.
- Am I creating a "similar but different" version? If yes, STOP.
- Should I modify the original instead? Usually yes.

**The Component Checklist (MANDATORY before creating):**

```
[ ] Searched codebase for similar components
[ ] Checked component library (shadcn, etc.) for existing solution
[ ] Verified no other component does 80%+ of what I need
[ ] Confirmed this will be reused (not a one-off)
[ ] Identified where this fits in the component hierarchy
```

**Examples of DIE HÖCHSTE VERRAT:**
```
VERBOTEN: Creating UserCard when ProfileCard exists and could be extended
VERBOTEN: Creating CustomButton when Button with variants exists
VERBOTEN: Creating FormInput when Input + wrapper pattern exists
VERBOTEN: Copy-pasting a component and "tweaking it a bit"
VERBOTEN: "This is slightly different so I'll make a new one"
```

**Correct approach:**
```
GUT: Extend existing component with new prop/variant
GUT: Use composition (slots) to customize behavior
GUT: Refactor original to support both use cases
GUT: Use the existing component even if not "perfect"
```

**The 80% Rule:** If an existing component does 80% of what you need, USE IT. Adapt your design to the component, not the other way around. Perfect is the enemy of done, and duplication is the enemy of maintainability.

---

## Core Principles

### 1. Logic Lives Outside the UI

**Extract main functionality to separate classes.**

Logic must be usable without knowing the UI exists. Services, not components, hold the brains. If your business logic requires importing Vue/React, you've failed.

```
BAD:  Component -> has logic -> renders
GOOD: Component -> calls service -> renders result
```

### 2. Simplicity Over Cleverness

**Preserve simple structures and stateless flows.**

No unnecessary complexity. Pure functions over stateful spaghetti. If you can't explain the data flow in one sentence, refactor until you can.

### 3. UI is Disposable

**Components exist to serve a purpose.**

Once data patterns become clear, restructure without hesitation. Don't get attached to markup. The component you wrote yesterday might be wrong today. Delete it.

### 4. Components Are Dumb Renderers

**Strict separation of concerns.**

| Components Do | Components Don't |
|---------------|------------------|
| Accept props | Make API calls |
| Render output | Hold business state |
| Manage ephemeral UI state | Contain business logic |
| Emit events | Transform data |

**Ephemeral UI state** (lives in component):
- `isModalOpen`, `hoveredId`, `filterText`, `expandedRows`

**Business state** (lives in store):
- `users`, `loading`, `error`, `pagination`

### 5. Simple One-Liners Stay Inline

**Don't extract trivial code.**

A simple reduce, basic filter, or `a/b*100` doesn't need a function. Only extract when logic is complex (nested loops, multi-step calculations, reusable algorithms).

### 6. Design System Variables Only — DEDUPLICATED AND STANDARDIZED

**All styling MUST use CSS variables from the design system. No exceptions.**

This is about more than consistency—it's about **single source of truth**. Every value in your CSS must trace back to a named variable in the design system.

**THE FOUR LAWS OF CSS VARIABLES:**

1. **DEDUPLICATED** — One variable per semantic concept. Not `--blue-500` AND `--primary-blue` AND `--button-bg`. ONE variable: `--color-primary`.

2. **STANDARDIZED** — Consistent naming conventions across the entire codebase. If spacing uses `--spacing-sm/md/lg`, you don't suddenly introduce `--gap-small`.

3. **CENTRALIZED** — All variables defined in ONE place. Not scattered across component files. Not duplicated in different stylesheets.

4. **THEME-AWARE** — All color variables MUST support light and dark themes. Theme switching must be instantaneous with zero component changes.

```css
/* VERBOTEN - arbitrary values */
.card {
  padding: 13px;
  color: #3a7bc8;
  border-radius: 6px;
  gap: 18px;
}

/* VERBOTEN - duplicate/inconsistent variables */
.card {
  padding: var(--card-padding);      /* Why does card have its own padding? */
  color: var(--blue-primary);        /* Inconsistent with --color-primary */
  border-radius: var(--border-rad);  /* Inconsistent with --radius-md */
}

/* GUT - standardized design system variables */
.card {
  padding: var(--spacing-md);
  color: var(--color-primary);
  border-radius: var(--radius-md);
  gap: var(--spacing-sm);
}
```

**Variable Audit Checklist:**

Before adding ANY CSS variable, answer:
- [ ] Does this variable already exist under a different name? → Use existing
- [ ] Does this follow the established naming pattern? → `--category-variant`
- [ ] Is this defined in the central design system file? → If not, add it there
- [ ] Could this be expressed with existing variables? → Combine them
- [ ] If it's a color, does it have both light AND dark theme values? → Required

**VERBOTEN patterns:**
```css
--card-padding: 16px;        /* Use --spacing-md instead */
--header-blue: #3a7bc8;      /* Use --color-primary instead */
--small-gap: 8px;            /* Use --spacing-sm instead */
--btn-radius: 4px;           /* Use --radius-sm instead */
```

**If you create a new variable, you must justify why existing variables don't work.**

---

**LAYOUT: FLEX + GAP, NEVER MARGINS**

**Margins are forbidden for layout spacing.** Use flex containers with gap instead.

Why margins are evil:
- Margins collapse unpredictably
- Margins create coupling between components (child knows about parent spacing)
- Margins make components non-portable (move component = broken spacing)
- Margins require mental overhead (margin-top vs margin-bottom, which component owns it?)

**The rule:** Parent containers own spacing via `gap`. Children never add margins for spacing.

```css
/* VERBOTEN - margins for layout */
.page-header {
  margin-bottom: 1rem;
}

.card {
  margin-top: 0.5rem;
}

.card + .card {
  margin-top: 1rem;  /* Lobotomized owl? More like lobotomized developer. */
}

/* GUT - parent owns spacing via gap */
.page-layout {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

/* Children have zero margin - they don't know about spacing */
.page-header { /* no margin */ }
.card { /* no margin */ }
```

**Gap variables must come from the design system:**
```css
gap: var(--spacing-xs);   /* 0.25rem */
gap: var(--spacing-sm);   /* 0.5rem */
gap: var(--spacing-md);   /* 1rem */
gap: var(--spacing-lg);   /* 1.5rem */
gap: var(--spacing-xl);   /* 2rem */
```

**The only acceptable margin use cases:**
- Negative margins for specific visual effects (overlapping elements)
- `margin: 0 auto` for centering
- Overriding third-party component styles (with `NATE-APPROVED` comment)

Everything else? Flex + gap.

---

**THEME ARCHITECTURE:**

The design system MUST support light and dark themes with instant switching. Structure:

```css
/* Base palette - raw color values (NEVER use directly in components) */
:root {
  --palette-gray-50: #f9fafb;
  --palette-gray-900: #111827;
  --palette-blue-500: #3b82f6;
  /* ... */
}

/* Semantic tokens - THESE are what components use */
:root, [data-theme="light"] {
  --color-bg: var(--palette-gray-50);
  --color-bg-elevated: white;
  --color-text: var(--palette-gray-900);
  --color-text-muted: var(--palette-gray-500);
  --color-border: var(--palette-gray-200);
  --color-primary: var(--palette-blue-500);
}

[data-theme="dark"] {
  --color-bg: var(--palette-gray-900);
  --color-bg-elevated: var(--palette-gray-800);
  --color-text: var(--palette-gray-50);
  --color-text-muted: var(--palette-gray-400);
  --color-border: var(--palette-gray-700);
  --color-primary: var(--palette-blue-400);
}
```

**Theme switching is ONE LINE:**
```javascript
document.documentElement.dataset.theme = 'dark';  // or 'light'
```

**VERBOTEN theme patterns:**
```css
/* VERBOTEN - hardcoded colors */
.card { background: white; }
.card.dark { background: #1f2937; }

/* VERBOTEN - conditional classes for theming */
<div :class="{ 'bg-white': !dark, 'bg-gray-800': dark }">

/* VERBOTEN - JavaScript color switching */
style.backgroundColor = isDark ? '#1f2937' : 'white';

/* GUT - semantic variable, theme handled by CSS */
.card { background: var(--color-bg-elevated); }
```

**Rules:**
- Components NEVER know what theme is active
- Components ONLY use semantic variables (`--color-bg`, not `--palette-gray-900`)
- Theme switch = change `data-theme` attribute = instant update, zero re-renders
- Palette variables are internal to the design system file only

---

**Tailwind is internal plumbing for component libraries only.**

Never use Tailwind utility classes directly in components (no `class="flex items-center p-4"`). Use component library components or CSS variables. Tailwind exists solely to power component library internals.

**Exception process:** If a custom arbitrary value is SPECIFICALLY REQUESTED by Nate, it must be documented:

```css
/* NATE-APPROVED: Custom spacing for legacy integration - 2026-02-05 */
.legacy-widget {
  margin-top: 13px;
}
```

No comment = no exception. If you see an arbitrary value without a `NATE-APPROVED` comment, it's a bug to be fixed.

### 7. Spacing Is Gap-Driven — No Margin Hacks

**Parent owns spacing. Children own internal padding. Nothing else.**

Spacing between sibling elements MUST be controlled by the parent container's `gap` property. Children never use `margin` to create space between themselves — that's the parent's job.

```css
/* VERBOTEN — margin on child to fake spacing */
.stats-panel {
  margin: var(--spacing-sm) var(--spacing-md);
  padding: var(--spacing-sm);
}

/* VERBOTEN — padding-top on a child to separate from sibling above */
.actions-bar {
  padding-top: var(--spacing-sm);
}

/* GUT — parent flex/grid with gap, children have zero external spacing */
.card-body {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm);
  padding: var(--spacing-md);
}

.stats-panel {
  /* internal padding only, no margin */
  padding: var(--spacing-sm) var(--spacing-md);
  background: color-mix(in srgb, var(--color-primary) 6%, transparent);
  border-radius: var(--radius-md);
}

.actions-bar {
  /* border-top for visual separation, gap handles the spacing */
  border-top: 1px solid var(--color-border-subtle);
}
```

**The pattern:**
1. Outer container (card, section, page) sets `padding` for inset
2. Flex/grid parent sets `gap` for spacing between children
3. Children set their own `padding` for internal content spacing
4. Children NEVER set `margin` — the parent gap handles it

**Why:** Margins collapse unpredictably, create invisible spacing that's hard to debug, and break when children are conditionally rendered (v-if removes an element but its margin logic doesn't adjust). Gap is explicit, predictable, and adapts automatically when children appear/disappear.

### 8. Never Swallow Errors

**All catch blocks MUST:**
- Log the error to console: `console.error('Context:', e)`
- Show user-facing error state (toast, error component, etc.)
- Never use empty `catch {}` blocks

The error message shown to user can be generic, but the console must have the real error.

### 9. Loading States Must Preserve Layout

**Loading skeletons must be 1:1 with the real component:**
- Same container elements, same grid/flex structure, same spacing
- Skeleton placeholders go WHERE the real content goes
- If real component has a table with 7 columns, skeleton has 7 skeleton cells
- NO generic "Loading..." spinners that cause layout shift
- User should see the EXACT shape of what's coming

**BAD:** `<div v-if="loading">Loading...</div>`
**BAD:** `<Skeleton v-if="loading" />` (single generic skeleton)
**GOOD:** Same markup as real component, but with `<Skeleton>` replacing text/images

---

## Architecture Patterns

### Operations vs Stores

| | Stores (Pinia/Redux) | Operations |
|---|---|---|
| **Owns** | Reactive state | Nothing (stateless) |
| **Holds** | `users`, `loading`, `error` | Pure functions only |
| **Does** | Cache data, catch errors, notify UI | Decide, orchestrate, combine |
| **Errors** | Catches and stores in `error` | Throws (lets bubble up) |

**Rules:**
- Single API call -> store calls API directly
- Multiple calls or conditional logic -> store delegates to operation
- All error handling happens in stores (operations throw)

### When to Use Operations

Operations are ONLY for pure logic. Ask: **Can this function run without any API calls?**

**YES - Valid operations:**
```typescript
// Computation from data
function calculateStats(items: Item[]): Stats { }

// Sorting logic
function sortByField(items: Item[], field: string): Item[] { }

// Building query params
function buildSearchParams(filters: Filters): Params { }

// Validation (display-side only)
function formatPhoneNumber(phone: string): string { }
```

**NO - Store does this:**
```typescript
// Just fetching - store does this
async function loadData() { return api.getData(); }

// Promise.all wrapper - store does this
async function initialLoad() { return Promise.all([api.getA(), api.getB()]); }
```

### Schema-First Development

Schemas are the single source of truth. Types are inferred, not duplicated.

```typescript
// Schema defines shape AND validation
export const UserSchema = z.object({
  id: z.number(),
  email: z.string(),
  active: z.boolean(),
});

// Type inferred - NOT a separate interface
export type User = z.infer<typeof UserSchema>;

// Parsing happens at API boundary
const response = await api.get('/users/1');
return UserSchema.parse(response.data);  // Validates at runtime
```

**Benefits:**
- Single source of truth (no type/schema drift)
- Runtime validation catches API changes
- Fixtures validate at load time (wrong mock = immediate failure)

### Response Shape Discipline

Backend returns consistent shapes. Frontend expects them.

| Method | Response Shape |
|--------|----------------|
| `list()` | `{ data: T[], pagination: {...} }` |
| `get(id)` | `{ success: true, data: T }` |
| Mutations | `{ success: true, data?: {...} }` |
| Errors | `{ success: false, error: "..." }` |

Parse the ENTIRE response at the boundary:
```typescript
const UserDetailResponse = ApiResponseSchema(UserSchema);

export const usersApi = {
  get: async (id: number) => {
    const response = await api.get(`/users/${id}`);
    return UserDetailResponse.parse(response);  // Validates wrapper + data
  },
};
```

---

## File Structure

```
src/
├── api/
│   ├── client.ts          # HTTP client + helpers
│   ├── users.ts           # API modules (one per domain)
│   └── index.ts
├── schemas/               # Zod schemas (single source of truth)
│   ├── common.ts          # Pagination, response wrappers
│   ├── user.ts
│   └── index.ts
├── operations/            # Pure logic only (when needed)
│   └── statistics.ts
├── stores/                # State management (own reactive state)
│   └── users.ts
└── components/            # UI components (render only)

tests/
├── setup.ts               # Mock adapter config
├── fixtures/              # Mock data (validates against schemas)
│   └── users.ts
└── operations/            # Pure logic tests only
    └── statistics.test.ts
```

---

## The Litmus Tests

**Is my logic in the right place?**
- Can I use this without Vue/React? -> If no, move to service/operation
- Is this business logic? -> If yes, it belongs in the backend (HOCHVERRAT!)

**Is my component doing too much?**
- Does it make API calls? -> Move to store
- Does it transform data? -> Move to operation
- Does it hold business state? -> Move to store

---

## Summary

```
SUPREME LAW: All real logic belongs in the backend. Violation = HOCHVERRAT.

Components  ->  Render props, emit events, ephemeral state only
Stores      ->  Call backend APIs, manage loading/error states
Operations  ->  Pure display formatting ONLY (when absolutely needed)
Schemas     ->  Single source of truth for types AND validation
```

**The goal:** Frontend is a thin integration layer. Backend owns all logic. The UI calls endpoints and displays results - nothing more.

**Remember:** If you're writing business logic in the frontend, you're committing HOCHVERRAT. Stop. Move it to the backend. Then call the endpoint.
