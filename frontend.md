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

### 6. Design System Variables Only

**All styling MUST use CSS variables from the design system.**

No arbitrary values. No magic numbers. No hardcoded colors. Everything comes from the design system.

```css
/* VERBOTEN - arbitrary values */
.card {
  padding: 13px;
  color: #3a7bc8;
  border-radius: 6px;
  gap: 18px;
}

/* GUT - design system variables */
.card {
  padding: var(--spacing-md);
  color: var(--color-primary);
  border-radius: var(--radius-md);
  gap: var(--spacing-sm);
}
```

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

### 7. Never Swallow Errors

**All catch blocks MUST:**
- Log the error to console: `console.error('Context:', e)`
- Show user-facing error state (toast, error component, etc.)
- Never use empty `catch {}` blocks

The error message shown to user can be generic, but the console must have the real error.

### 8. Loading States Must Preserve Layout

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
