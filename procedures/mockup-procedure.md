# Mockup Procedure

> How to build mockups that don't create double work.

---

## The Problem

Monolithic page mockups (one big HTML file) lead to:
- Building the UI twice: once in static HTML, again in Vue
- CSS written twice with diverging values
- Pixel-matching hell when the two inevitably drift
- No clear mapping between mockup sections and real components

---

## The Rule

**Mockups are built per-component, not per-page. Each mockup file imports the real `design-system.css`.**

The mockup IS the component spec. When you build the Vue component, you copy the HTML structure and class names. The CSS is already done — same tokens, same file.

---

## Structure

```
static/mockups/
  archive/                     ← beloved mockups that survived review (MAIN REPO ONLY, survives worktree deletion)
  components/
    detail-header.html        ← one component per file
    action-bar.html
    progress-bar.html
    viewing-section.html
    doc-checklist.html
    section-card.html
  compositions/
    application-detail.html   ← page-level: assembles component mockups
    client-detail.html
  _shared.css                 ← minimal mockup scaffolding (body bg, layout helpers)
```

Each component mockup is a self-contained HTML file:

```html
<!DOCTYPE html>
<html data-theme="dark">
<head>
  <link rel="stylesheet" href="../../assets/css/design-system.css">
  <link rel="stylesheet" href="../_shared.css">
  <style>
    /* Component-scoped styles — these become <style scoped> in Vue */
    .detail-header { ... }
    .profile-avatar { ... }
  </style>
</head>
<body>
  <div class="mockup-container" style="width: 800px;">
    <!-- Exactly what the Vue template will look like -->
    <div class="detail-header">
      <div class="profile-avatar">SJ</div>
      <div class="profile-info">
        <span class="profile-name">Sarah Jones</span>
        <span class="profile-meta">Applied Jan 21, 2025</span>
      </div>
    </div>
  </div>
</body>
</html>
```

The `_shared.css` file is minimal — just body background, centering, and a `.mockup-container` wrapper:

```css
body {
  margin: 0;
  padding: 2rem;
  background: var(--color-bg);
  font-family: var(--font-family);
  color: var(--color-text);
}

.mockup-container {
  margin: 0 auto;
}
```

---

## Workflow

### Phase 1: Design (mockup)

1. Create `static/mockups/components/my-component.html`
2. Import the real `design-system.css`
3. Write HTML structure + scoped CSS using design tokens
4. Open in browser, iterate until it looks right
5. Serve on port 8888: `python -m http.server 8888 -d static/mockups`

### Phase 2: Build (Vue component)

1. Create `components/my-component.vue`
2. Copy the HTML structure into `<template>` — add `v-for`, `:class`, etc.
3. Copy the scoped CSS into `<style scoped>` — verbatim, it already uses design tokens
4. Add `<script setup>` with props, emits, computeds
5. Done. CSS is identical because it came from the same source.

### Phase 3: Verify (comparison)

1. Open mockup HTML in browser, screenshot the component
2. Render the Vue component with equivalent data, screenshot
3. Compare with ImageMagick: `magick compare mockup.png real.png diff.png`
4. Differences should be near-zero since the CSS is shared

---

## Composition Mockups

Page-level mockups go in `compositions/`. These assemble component mockups to test layout:

```html
<!DOCTYPE html>
<html data-theme="dark">
<head>
  <link rel="stylesheet" href="../../assets/css/design-system.css">
  <link rel="stylesheet" href="../_shared.css">
  <style>
    /* Page layout styles */
    .detail-grid { display: grid; grid-template-columns: 1fr 1fr; gap: var(--spacing-lg); }
    .detail-col { display: flex; flex-direction: column; gap: var(--spacing-lg); }

    /* Include component styles inline or via separate CSS files */
    /* ... */
  </style>
</head>
<body>
  <div class="mockup-container" style="width: 1200px;">
    <!-- Assemble components to test page layout -->
    <div class="detail-header">...</div>
    <div class="detail-grid">
      <div class="detail-col">
        <div class="section-card">...</div>
        <div class="section-card">...</div>
      </div>
      <div class="detail-col">
        <div class="section-card chat-card">...</div>
      </div>
    </div>
  </div>
</body>
</html>
```

Use these to validate that components work together before touching Vue. But don't over-invest — the component mockups are the real deliverable.

---

## DO's and DON'Ts

### DO

- **Import the real `design-system.css`** — same tokens, same source of truth
- **One component per mockup file** — isolation prevents monolithic drift
- **Use the exact class names you'll use in Vue** — copy-paste should work
- **Test dark AND light themes** — flip `data-theme` attribute
- **Keep sample data realistic** — use real-ish names, addresses, prices
- **Iterate on the mockup first** — cheaper than iterating in Vue

### DON'T

- **Don't build monolithic page mockups** — they become unmaintainable and create double work
- **Don't use hardcoded colors/sizes** — if it's not a design token, it doesn't exist
- **Don't create mockup-specific CSS variables** — use what the design system provides
- **Don't skip the mockup** — "I'll just build it in Vue" leads to aimless iteration
- **Don't keep stale mockups** — if the Vue component changed, update or delete the mockup

---

## When to Mockup

**ALWAYS mockup when:**
- Designing a new component that doesn't exist yet
- Redesigning an existing component's layout
- Exploring multiple layout Ausführungen (ausfuhrung-a, ausfuhrung-b)
- Building something visually complex (grids, multi-section layouts)

**SKIP the mockup when:**
- Adding a prop to an existing component
- Fixing a CSS bug
- Changing text/labels
- The component is trivially simple (single button, icon swap)

---

## Summary

```
1. Mockups are per-component, not per-page
2. Every mockup imports the real design-system.css
3. CSS written in the mockup IS the CSS used in Vue (copy-paste)
4. Composition mockups test page layout by assembling components
5. Compare mockup vs real with ImageMagick to verify parity
```

**The goal:** Write CSS once in the mockup, copy it to Vue. No double work. No drift. No pixel-matching nightmares.
