# Lessons Learned

> Hard-won rules from repeated mistakes. Read this before every session. If a lesson here contradicts your instinct, the lesson wins.

---

### 2026-02-08 — Claimed CSS fix was done without visual proof
**What happened**: Said subscriptions page spacing was fixed. It wasn't — dynamic CSS was removing styles after page load, and nobody checked.
**Root cause**: Lazy claim of completion without verification.
**Prevention rule**: **NEVER claim a CSS/layout change is done without a screenshot proving it.** Take the screenshot AFTER the page loads (not during skeleton/loading). View it yourself. If you didn't see it with your own eyes, it's not done.

---

### 2026-02-08 — Used wrong default repo for browser agents
**What happened**: Set up agent worktrees in `agent_dashboard_frontend` instead of `agent-dashboard-vue`. Had to fix in 3 places (CLAUDE.md, MEMORY.md, SKILL.md).
**Root cause**: Assumed wrong repo without checking documentation.
**Prevention rule**: Default agent repo is **agent-dashboard-vue**. Not agent_dashboard_frontend. Check before acting.

---

### 2026-02-08 — Child margins fought parent flex gap, doubled spacing
**What happened**: TabHeader had `margin-bottom: 1rem` AND the parent had `gap: 1.5rem`. Spacing was wrong because both applied. Removed gap thinking margin was right — wrong answer.
**Root cause**: Not understanding that parent owns spacing, children don't.
**Prevention rule**: **Parent containers own ALL spacing via `gap`. Children have ZERO margins.** When spacing looks wrong, check for margin/gap conflicts. Remove the child margin, never the parent gap.

---

### 2026-02-15 — Same overflow bug in 3 different dashboard widgets
**What happened**: Response queue, messages widget, and listings attention widget all overflowed their containers. Same root cause, same fix needed 3 times.
**Root cause**: Flex children don't shrink below their content size by default.
**Prevention rule**: **Scrollable flex children need 3 properties: `overflow-y: auto`, `flex: 1`, `min-height: 0`.** The `min-height: 0` is the one you'll forget — without it, the flex item refuses to shrink and overflows its parent. Check EVERY scrollable container inside a flex parent.

---

### Ongoing — Telling Nate to do things instead of doing them
**What happened**: Saying "you should run X" or "please restart Y" instead of just doing it.
**Root cause**: Defaulting to advisory mode instead of action mode.
**Prevention rule**: **If you can run a command, edit a file, or start a process — DO IT.** Don't ask, don't suggest, don't explain. Just execute. Exceptions: services listed in "DO NOT TOUCH", GUI-only actions, destructive operations needing explicit confirmation.

---

### Ongoing — Not searching for existing solutions before building
**What happened**: Started building custom tools/features that already exist as open source.
**Root cause**: Builder instinct overriding research discipline.
**Prevention rule**: **Search GitHub, npm, and the web FIRST.** If an existing solution does 80%+ of what's needed, use it. Only build custom when nothing exists or existing solutions are fundamentally wrong for the use case.

---

### Ongoing — Arbitrary CSS values instead of design system variables
**What happened**: Writing `padding: 13px` or `color: #3a7bc8` instead of using `var(--spacing-md)` or `var(--color-primary)`.
**Root cause**: Taking shortcuts, not checking the design system.
**Prevention rule**: **ALL styling uses CSS variables.** No hardcoded pixels, no hex colors, no magic numbers. If a value doesn't have a variable, add it to the design system — don't inline it. Any arbitrary value without a `NATE-APPROVED` comment is a bug.

---

### Ongoing — Using margins for layout spacing
**What happened**: Adding `margin-bottom` or `margin-top` to create spacing between elements.
**Root cause**: Old CSS habits.
**Prevention rule**: **Flex + gap for ALL layout spacing.** Margins collapse unpredictably, create coupling between components, and make them non-portable. Parent owns spacing via `gap`, children never add margins. Only exceptions: negative margins for visual effects, `margin: 0 auto` for centering, third-party overrides with `NATE-APPROVED`.

---

### Ongoing — Direct Tailwind utility classes in components
**What happened**: Writing `class="flex items-center p-4 text-sm"` directly in component templates.
**Root cause**: Tailwind is installed, so it feels available.
**Prevention rule**: **Tailwind is internal plumbing for shadcn-vue ONLY.** Never use utility classes directly. Use shadcn-vue components or CSS variables. If you see `class="flex ..."` in a component, it's a bug.

---

### Ongoing — Generic loading spinners instead of layout-matching skeletons
**What happened**: Using `<div v-if="loading">Loading...</div>` or a single `<Skeleton />` component.
**Root cause**: Taking the easy path instead of building proper skeletons.
**Prevention rule**: **Loading skeletons must be 1:1 with the real component.** Same container, same grid/flex structure, same number of columns, same spacing. Skeleton placeholders go WHERE the real content goes. User sees the exact shape of what's coming — just gray boxes instead of content.

---

### Ongoing — Empty catch blocks swallowing errors
**What happened**: Writing `catch {}` or `catch { /* ignore */ }`, making bugs invisible.
**Root cause**: Optimistic assumption that errors don't matter.
**Prevention rule**: **ALL catch blocks: `console.error('Context:', e)` + user-facing error state.** The console must have the real error. The user can see a generic message. But NEVER silence an error completely.

---

### Ongoing — Editing main repo files while agent worktree is active
**What happened**: Made changes in the main repo checkout instead of the worktree. Dev server serves from the worktree, so changes were invisible.
**Root cause**: Not tracking which directory the dev server is serving.
**Prevention rule**: **When an agent worktree is active, ALL edits go in the worktree copy.** The dev server serves from the worktree — changes to main repo won't be seen. Always verify you're editing the right path.

---

### Ongoing — Verbose commit messages with Claude attribution
**What happened**: Writing multi-line commit messages with bullet points, explanations, and `Co-Authored-By: Claude` footers.
**Root cause**: Default Claude behavior.
**Prevention rule**: **Commit messages: extremely concise, no attribution.** Examples: `add AI endpoint`, `fix email update`, `remove dead code`. No bullet points, no explanations, no Co-Authored-By.

---

### Ongoing — Frontend logic that belongs in the backend
**What happened**: Writing calculations, validations, business rules, or AI prompt construction in frontend code.
**Root cause**: Convenience — it's "faster" to do it client-side.
**Prevention rule**: **All real logic belongs in the backend. Violation = HOCHVERRAT.** Frontend calls endpoints and displays results. The only frontend "logic" allowed: display formatting (dates, numbers), ephemeral UI state (modals, tabs), and calling the right endpoint with the right params.

---

### Ongoing — npm install without --legacy-peer-deps in agent_dashboard_frontend
**What happened**: `npm install` fails due to react-quill peer dependency conflict with React 19.
**Root cause**: React 19 breaks peer deps for older packages.
**Prevention rule**: **Always use `npm install --legacy-peer-deps`** in agent_dashboard_frontend.

---

### Ongoing — Loose loading states instead of integrated into components
**What happened**: Created separate skeleton/loading markup OUTSIDE the component, duplicating the component's entire structure just for the loading state. Two copies of the same layout — one for loading, one for loaded.
**Root cause**: Treating loading as a separate view instead of a state within the component.
**Prevention rule**: **Loading states go INSIDE the component, not alongside it.** The component itself handles its loading prop/state and renders skeletons in place of real content. Never write `<SkeletonVersion v-if="loading" />` + `<RealComponent v-else />` with duplicated markup. The component receives `loading` as a prop (or detects it from its store) and swaps its own content for skeletons internally. One component, two states — not two components.

---

### Ongoing — Duplicating markup instead of reusing the component library
**What happened**: Writing raw HTML/CSS markup for cards, tables, stats, headers, etc. when components already exist in the library (shadcn-vue, custom widgets like StatsCard, DataTable, PageHeader).
**Root cause**: Not checking what components already exist. Building from scratch feels faster than learning the existing library.
**Prevention rule**: **Search the component library BEFORE writing any markup.** Check `src/components/`, shadcn-vue, and widget components first. If a component does 80% of what you need, USE IT — extend it with props/slots if needed. Writing raw `<div class="card">` when `<Card>` exists is duplication. Writing a custom table when `<DataTable>` exists is duplication. Every piece of hand-written markup is a liability if a component already handles it.

---

### Ongoing — Agent dev servers without NO_HMR=true
**What happened**: HMR WebSocket connections interfere with browser automation (agent-browser CDP).
**Root cause**: Default dev server enables HMR.
**Prevention rule**: **NO_HMR=true is mandatory** when starting dev servers for browser testing agents.
