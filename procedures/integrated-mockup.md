# Integrated Mockup Procedure

> Prototype UI inside the real app. No static HTML. No re-integration.

---

## The Problem

Static HTML mockups create a translation step: HTML → Vue. That translation is where things break. Class names get mangled, structure drifts, interactions are missing, and the "integration" becomes a full rebuild.

---

## The Rule

**Mockups are real Vue components in their final location. The only fake thing is the data.**

The component is born production-ready: correct file path, correct props interface, correct store wiring. When the backend catches up, you swap hardcoded data for API calls. The component never changes.

---

## How It Works

### Components — Final Location From Day One

```
# DON'T create mockup files
static/mockups/components/detail-header.html     ← NO

# DO create the real component
src/components/listings/DetailHeader.vue          ← YES
```

The component uses real design system tokens, real Naive UI primitives, real slots and props. It IS the final component — just fed fake data.

### Stores — Where Fake Data Lives

Fake data lives in the store, never in the component. Components are dumb renderers — they don't know or care whether data is fake or real.

```javascript
// stores/listingDetail.js — MOCKUP PHASE
// TODO(mockup): Replace with API call when endpoint is ready
const getListingDetail = async (id) => {
  return {
    title: 'Altbauwohnung mit Balkon',
    price: 285000,
    rooms: 3,
    area: 78.5,
    address: 'Margaretenstraße 42, 1050 Wien',
    images: ['/dev/placeholder-1.jpg', '/dev/placeholder-2.jpg'],
    seller: { name: 'Maria Huber', phone: '+43 660 1234567' },
  }
}
```

```javascript
// stores/listingDetail.js — CONNECTED PHASE
// Just replace the function body. Store interface unchanged. Component unchanged.
const getListingDetail = async (id) => {
  const { data } = await request.get(`/listings/${id}`)
  return data
}
```

The `TODO(mockup)` marker makes fake data grep-able. Before any release: `grep -r "TODO(mockup)"` — if anything comes back, it's not done.

### The Component Doesn't Know

```vue
<!-- components/listings/DetailHeader.vue -->
<!-- Identical in mockup phase and production. It never changes. -->
<template>
  <div class="detail-header">
    <h1 class="detail-title">{{ listing.title }}</h1>
    <span class="detail-price">{{ formatPrice(listing.price) }}</span>
    <span class="detail-meta">{{ listing.rooms }} Zimmer · {{ listing.area }} m²</span>
  </div>
</template>

<script setup>
defineProps({
  listing: { type: Object, required: true }
})
</script>
```

It accepts props. It renders. It doesn't care where the data came from.

---

## Workflow

### Phase 1: Build (mockup with fake data)

1. Create the component in its final path
2. Wire it into the real route/page where it will live
3. Store returns hardcoded fake data, marked with `TODO(mockup)`
4. Iterate on layout, interactions, animations with hot reload
5. Test light and dark themes
6. Explore alternatives — swap fake data shapes, try different layouts

### Phase 2: Connect (replace fake with real)

1. Backend endpoint is ready
2. Replace hardcoded store data with API call
3. Component is untouched — it already renders the right shape
4. Test with real data, fix any edge cases (empty states, long strings, etc.)

### Phase 3: Clean (remove all traces)

1. `grep -r "TODO(mockup)"` — nothing should come back
2. Delete any placeholder assets (`/dev/placeholder-*.jpg`)
3. No fake data remains anywhere in the codebase

---

## Multi-Variant Exploration

Don't settle on the first idea. Build multiple variants, show them all simultaneously in the browser, let the user compare.

The user decides what to explore. Could be layouts, component ideas, interaction patterns, page structures, navigation approaches — anything. You build what they ask for, show the options, they pick.

### How It Works

1. **User says what to explore** — "show me 3 ways to do the listing detail page" or "try different sidebar ideas" or "explore card vs table for this data"
2. **Build variants SEQUENTIALLY, ONE AT A TIME** — build A, screenshot it, then build B deliberately different from A, screenshot it, then build C different from both. Each variant is informed by the previous ones.
3. **Show via agent-browser** — screenshots or live tabs, user compares
4. **User picks** — winner stays, everything else deleted

### SEQUENTIAL IS NON-NEGOTIABLE

**NEVER build variants in parallel.** The entire point of multiple variants is deliberate divergence. If you dispatch 3 agents in parallel, they can't see each other's work and will converge on similar ideas.

The sequence matters:
1. Build variant A. Screenshot it. Understand what it does well and what it doesn't.
2. Build variant B. It MUST be deliberately different from A — different layout, different visual approach, different interaction pattern. You know what A looks like, so you can ensure B goes a different direction.
3. Build variant C. Different from BOTH A and B. You've seen both, so you know what's been explored and what hasn't.

**This is not parallelizable. Do not try.**

### Variant Files

Variants are suffixed with `.A`, `.B`, `.C` and live next to where the final component will go:

```
# Exploring component layouts:
src/components/listings/
  DetailHeader.A.vue
  DetailHeader.B.vue
  DetailHeader.C.vue

# Exploring page structures:
src/pages/
  ListingDetail.A.vue
  ListingDetail.B.vue

# Exploring completely different component ideas:
src/components/dashboard/
  ActivityFeed.A.vue        ← timeline approach
  ActivityFeed.B.vue        ← card grid approach
  ActivityFeed.C.vue        ← compact table approach
```

### Showcase Page

A throwaway page that imports all variants into tabs:

```vue
<!-- pages/dev/mockup-showcase.vue -->
<template>
  <n-tabs type="card">
    <n-tab-pane name="A" tab="A — Timeline">
      <ActivityFeedA v-bind="fakeProps" />
    </n-tab-pane>
    <n-tab-pane name="B" tab="B — Card Grid">
      <ActivityFeedB v-bind="fakeProps" />
    </n-tab-pane>
    <n-tab-pane name="C" tab="C — Compact Table">
      <ActivityFeedC v-bind="fakeProps" />
    </n-tab-pane>
  </n-tabs>
</template>
```

Serve on 8888. Open via `agent-browser`. Screenshot each tab.

```bash
agent-browser open http://localhost:8888/dev/mockup-showcase
agent-browser screenshot
# click tab B, screenshot, click tab C, screenshot
```

### After the Pick

1. **Winner gets renamed** — `ActivityFeed.B.vue` → `ActivityFeed.vue`
2. **Losers deleted** — `.A.vue`, `.C.vue` gone
3. **Showcase page deleted**
4. **Clean commit** — only the winner remains

---

## Fake Data Rules

### DO

- **Keep fake data realistic** — real-ish names, addresses, prices in the right locale
- **Match the expected API shape exactly** — same field names, same nesting, same types
- **Mark every fake data source** — `TODO(mockup)` comment, no exceptions
- **Put fake data in stores only** — components never hold fake data
- **Include edge cases** — empty arrays, long strings, null optionals — test these during mockup phase

### DON'T

- **Don't fake data in components** — components are dumb renderers, they don't know about data sources
- **Don't create separate fixture files** — fake data lives inline in the store function it replaces
- **Don't invent API shapes** — if the backend schema exists, match it. If it doesn't, agree on the shape first
- **Don't leave fake data behind** — `TODO(mockup)` grep must return zero before shipping

---

## When to Use This

**USE integrated mockup when:**
- Building a new view or component that needs interaction design
- Exploring layout alternatives within the live app
- Prototyping a flow (multi-step forms, wizards, transitions)
- Backend isn't ready yet but frontend work can start

**DON'T USE when:**
- Fixing a CSS bug (just fix it)
- Adding a prop to an existing component (just add it)
- The backend endpoint already exists (just connect it)
- Trivially simple component (single button, label change)

---

## Summary

```
1. Component lives in its final path from day one
2. Store returns fake data marked with TODO(mockup)
3. Component is a dumb renderer — doesn't know data is fake
4. When backend is ready, swap store internals — component untouched
5. grep -r "TODO(mockup)" = 0 results before shipping
```

**The goal:** Zero re-integration. The mockup IS the production component. Only the data source changes.
