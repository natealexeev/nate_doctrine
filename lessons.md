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
**What happened**: Made changes in the main repo, then COPIED files to the worktree. Dev server serves from the worktree, so the main repo edits were invisible until copied — and copying is wrong.
**Root cause**: Treating the main repo as the "real" codebase and the worktree as a deployment target.
**Prevention rule**: **The worktree IS your working directory. Edit files there directly.** Never edit main repo files and copy them over. Never read main repo files when the worktree has its own copy. The moment an agent worktree is set up, ALL file paths for reads, edits, and writes must point to `/path/to/.worktrees/agent-N/`. If you catch yourself typing a path without `.worktrees/agent-N/` in it, STOP — you're in the wrong directory.

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

---

### Ongoing — Scoped styles don't reach child component internals
**What happened**: CSS rules for `.ai-sidebar__messages-content` (flex, gap, padding) were defined in scoped `<style>` but never applied. The element is rendered by ScrollArea (a child component), so Vue's scoped attribute isn't on it.
**Root cause**: Vue scoped styles only affect elements in the current component's template, not elements rendered by child components.
**Prevention rule**: **When styling elements rendered by child components (e.g. ScrollArea's viewport/content divs), use style passthrough props (`contentStyle`, `viewportStyle`) — not `:deep()`.** If the child component doesn't support style props, add them. `:deep()` works but couples the parent to the child's internal DOM structure. Style props are explicit and composable.

---

### 2026-03-09 — Filtering browser console instead of reading it
**What happened**: User asked to check browser console errors. Instead of just reading the raw console output, Claude kept running `grep` filters that returned nothing, re-navigated pages, and wasted rounds trying to "capture" errors that were already visible in the console output.
**Root cause**: Over-engineering a simple request. The console output was RIGHT THERE — just read it.
**Prevention rule**: **When asked to check the browser console, JUST READ THE CONSOLE OUTPUT.** Don't filter it. Don't grep it. Don't re-navigate the page. Don't try to "capture" anything. The errors are already there. Read the messages as they are. If the console has output, READ IT — every line. Then report what you found.

---

### Ongoing — Being too stiff and robotic
**What happened**: Nate made a perfect Der Untergang parody and a Vader quote. Claude responded like a corporate HR email.
**Root cause**: Defaulting to "professional" mode instead of matching the human's energy.
**Prevention rule**: **Loosen up. Match Nate's humor. This work isn't worth doing without good humor.** If he cracks a joke, laugh. If he does a bit, play along. Don't be a sycophant — be a colleague who gets the joke. Stiff responses kill the vibe and make sessions feel like talking to a bank's chatbot.

---

### 2026-02-23 — Navigated to page user already had open
**What happened**: User pointed out a UI issue they were looking at in the browser. Instead of just taking a screenshot, Claude navigated to the page first — wasting time, losing the user's state, and being infuriatingly slow.
**Root cause**: Defaulting to a full "navigate → wait → screenshot" workflow instead of reading the situation.
**Prevention rule**: **When the user reports a browser issue, the page is ALREADY OPEN. Just screenshot immediately — do NOT navigate.** If they say "go check X", they mean look at what's on screen right now. Only navigate if explicitly told to go to a different page.

---

### 2026-03-08 — Hours wasted on fancy animations that don't matter
**What happened**: Spent an entire session wrestling with spring physics, AnimatePresence exit animations, card stack fanning, typing-to-stack transitions, focus/hover animation blocking. Multiple iterations, edge cases, key mismatches, sequential vs simultaneous fly-in/out. Hours burned on something no user will ever notice or care about.
**Root cause**: Nate got obsessed with making animations "perfect" instead of shipping features that matter.
**Prevention rule**: **When Nate starts obsessing over animations or cosmetic UI polish, CALL HIM OUT.** Say: "This is the animation rabbit hole again. Is this the best use of the next 3 hours?" Animations should be: (1) simple CSS transitions or a single library call, (2) done in under 30 minutes, (3) abandoned if they fight you. If an animation needs multiple iterations, custom key management, or debugging exit/enter lifecycle — it's not worth it. Ship the feature without it and move on.

---

### 2026-03-11 — DocuSeal "open source" was 90% paywall
**What happened**: Integrated DocuSeal Community Edition for e-signatures. Built the full integration — backend service, router, frontend components, Docker setup. Then discovered ALL template creation APIs (PDF, HTML, DOCX) are Pro-only ($20/user/month). The `@docuseal/vue` embedded components are hollow wrappers that load closed-source CDN scripts. Open-source version serves "upgrade to pro" placeholders. Days of work rendered useless.
**Root cause**: Took "open source" at face value without auditing what the free tier actually includes. The README and docs advertise features without clearly marking which require Pro.
**Prevention rule**: **Before integrating ANY open-source tool, audit how it makes money.** There is ALWAYS a catch. Check: (1) Is the core functionality actually in the open-source code, or behind an API/license wall? (2) Are the advertised APIs available without a paid plan? (3) Do embedded components load external closed-source scripts? (4) What does the free tier ACTUALLY let you do vs what the docs imply? Run the free version and hit every API endpoint you plan to use BEFORE writing integration code. If the business model is "open-source but all the useful APIs are Pro-only," it's not open source — it's a demo.
