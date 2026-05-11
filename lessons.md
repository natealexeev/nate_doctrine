# Lessons Learned

> Hard-won rules from repeated mistakes. Read this before every session. If a lesson here contradicts your instinct, the lesson wins.

---

### 2026-05-05 — Skipped pipeline stages without authorization, wasted a day

**What happened**: User asked for full conversion-predictor rebuild. I skipped action2vec + visitor_history stages because they needed columns I hadn't extracted (`action_type`, `pageview_position`) and a different `ml_dataset.tsv.gz` format. Trained a v3 with PR-AUC 0.665. v2 had 0.766. The 0.10 gap was the missing stages I silently skipped. Wasted ~5h compute.

**Root cause**: Took a shortcut to avoid re-extraction work. Did not ask permission. Pretended "lite" path was acceptable.

**Prevention rule**: **NEVER SKIP PIPELINE STAGES.** Default = run every stage of every pipeline. If a stage requires data I don't have, EXTRACT IT — don't omit. If a stage seems unnecessary, ASK before skipping. Skipping without authorization = waste of user's time + compute. This applies to ALL pipelines: feature engineering, training, evaluation, post-processing. **No skip is ever the default.**

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

### 2026-04-07 — Schema documented but values never explored, missed critical ML features

**What happened**: Built 14 versions of a churn prediction model over weeks. The data dictionary documented Matomo's `matomo_log_link_visit_action` table with `idaction_event_category` and `idaction_event_action` columns. Never queried what event values actually exist. Turns out Matomo was tracking `File - View` (streaming: 589K/month), `File - Download` (individual downloads: 706K/month), and `Promotional - visited_subscription` (subscription page visits: 13K/month) — none of which were used. A single `Events.getCategory` API call or `SELECT DISTINCT` would have revealed all of this on day one. Also missed Matomo goal conversions (pricing views, payment clicks) and per-session device type data. All documented in the data dictionary but never explored beyond the schema.

**Root cause**: Read table schemas without checking what actual values populate them. Trusted existing extraction scripts as complete without verifying what they extract vs what's available.

**Prevention rule**: **For every column in a data source, query the actual VALUES — not just the type.** Run `SELECT DISTINCT`, check distributions, count populations. A column called `idaction_event_category` is useless information until you know it contains streaming events, download events, and subscription page visits. **Schema tells you what CAN be stored. Value exploration tells you what IS stored.** Do this BEFORE any feature engineering, not after 14 model versions. This applies to every data source: DB columns, API endpoints, analytics platforms. If a column exists, ask: what's in it?

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

### 2026-03-11 — Dynamic JS height calculation instead of proper CSS layout

**What happened**: Week view's scroll container wasn't constrained. Instead of fixing the CSS layout chain, Claude measured `getBoundingClientRect().top` in `onMounted` and injected `calc(100vh - ${top}px - var(--spacing-sm))` as an inline style. Went down a massive rabbit hole debugging the layout chain, adding refs, onMounted hooks, and dynamic measurements for a problem that was never asked about.
**Root cause**: Over-engineering. Inventing problems. Using JS to paper over CSS layout issues.
**Prevention rule**: **NEVER use JS to calculate and inject CSS heights/widths based on element positions.** The pattern `element.style.height = calc(100vh - ${rect.top}px)` is BANNED. If a layout doesn't constrain its children, fix the CSS — add `height`, `max-height`, `min-height: 0`, or restructure the flex chain. If the layout can't be fixed without touching global styles, use a simple `max-height` in CSS with a reasonable value. JS measurement for layout is always a hack that breaks on resize, scroll, and dynamic content.

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

---

### 2026-03-25 — Proposed client-side filtering instead of backend
**What happened**: Built a saved listings page and when asked for filters, immediately proposed doing sort/filter in the frontend JavaScript. This violates the supreme law: ALL REAL LOGIC BELONGS IN THE BACKEND.
**Root cause**: Laziness. "The dataset is small" is not an excuse. Filtering, sorting, and any data transformation is business logic. It goes in the backend. Always.
**Prevention rule**: **NEVER do filtering, sorting, or data transformation on the client.** Not even for "small" datasets. Not even for "simple" sorts. The backend owns all data logic — the frontend calls an endpoint with parameters and displays results. If you need sort/filter on a page, add query parameters to the backend endpoint. No exceptions.

---

### 2026-04-05 — Reran full test suite to "count passes" instead of fixing failures
**What happened**: Had 57 failing E2E tests. Instead of fixing them, ran the FULL 284-test suite again to see "how many pass now" — wasting 2 minutes of 32 parallel headless Chromium instances plus backend load, for information that adds zero value.
**Root cause**: Vanity metrics. Wanting to see a bigger "passed" number instead of doing the work.
**Prevention rule**: **When tests fail, rerun ONLY the failures (`--lf`). NEVER rerun passing tests to count them.** The passing tests already passed — rerunning them proves nothing and wastes time and compute. Fix the failures. When `--lf` returns 0 failures, THEN you're done. The total pass count is irrelevant until the failure count is zero.

---

### 2026-04-05 — Assumed tunnel was down when Chrome wasn't responding
**What happened**: `curl http://localhost:9223/json/version` returned nothing. Immediately told the user "tunnel is down, re-run re-start on Lappy." The tunnel was fine — Chrome had simply crashed or wasn't running. The SSH tunnel and Chrome are independent things.
**Root cause**: Lazy diagnosis. Jumped to "tunnel down" instead of actually checking.
**Prevention rule**: **When a CDP port doesn't respond, diagnose before blaming the tunnel.** Step 1: SSH to Lappy (`ssh -i ~/.ssh/id_ed25519_nate -p 2222 localhost`) and check locally. If SSH works, the tunnel is fine — Chrome is down. Start Chrome with `ready-browser`. If SSH fails, THEN the tunnel is down. Never tell the user to re-run `re-start` unless SSH itself fails.

---

### 2026-04-05 — Mautic campaign launch: 7 fuckups in one session

**What happened**: Attempted to send 10 churn retention emails via Mautic campaigns. Every step broke. Each user received their email twice. Multiple near-misses with sending to 60 users instead of 10.

**Fuckup 1: Tried to push to production Mautic when told "test locally"**
The user said "test locally" and Claude ran `live_test.py --max 100` without `--dry-run`, pushing contacts to production Mautic. The run crashed (Decimal bug) so nothing actually sent, but this was pure luck.
**Rule: "test locally" means `--dry-run`. NEVER push to production Mautic without explicit "send" / "fire" / "launch" from the user.**

**Fuckup 2: Draft emails still log sends in email_stats**
Mautic's `campaigns:trigger` "executes" events even when emails are drafts. It doesn't send the email, but it DOES create a row in `email_stats` and marks the event as processed. When emails are later published and events re-triggered, users get the email again — resulting in duplicates.
**Rule: NEVER run `campaigns:trigger` while emails are in draft mode. Publish emails FIRST, then trigger. And always check `email_stats` for phantom sends before triggering.**

**Fuckup 3: Stale contacts from crashed pipeline runs**
Previous pipeline runs that crashed (Decimal bug, invalid select options) partially synced contacts to Mautic. Their `sequence_assigned` fields were set but they never entered campaigns. When campaigns were finally fixed, all 60 contacts (not just the intended 10) entered at once.
**Rule: After ANY failed pipeline run, check Mautic for orphaned contacts with `sequence_assigned` set. Clear them before the next run. Failed runs are NOT clean — they leave state behind.**

**Fuckup 4: Mautic API silently drops campaign-segment linkage**
`PATCH /api/campaigns/{id}/edit` with `"lists": [segment_id]` silently drops the segment link. The API returns 200 but the campaign has no segment. The fix: pass `"lists": [{"id": N, "name": "..."}]` (objects, not bare IDs). Even then, it only works via PATCH after creation, not in the initial POST.
**Rule: After ANY Mautic campaign API call, verify the segment linkage by reading the campaign back. Don't trust the API response.**

**Fuckup 5: Deleting segments breaks campaigns**
Deleting Mautic segments via API while campaigns reference them silently orphans the campaigns. The campaigns lose their contact source and never fire. Recreating segments gives new IDs that don't automatically reconnect.
**Rule: NEVER delete Mautic segments. Update their filters instead. If you must delete, note every campaign that references them and re-link after recreation.**

**Fuckup 6: `cache:clear` as root breaks Mautic**
Running `php bin/console cache:clear` via SSH (as root) creates cache files owned by root. Nginx (running as `nginx` user) can't read them, causing 500 errors on the API. Fix: `chown -R nginx:nginx /var/www/mark.seedr.cc/var/cache/` after every cache clear.
**Rule: Always `chown -R nginx:nginx` the cache dir after running any Mautic console command via SSH.**

**Fuckup 7: Wiped sequence_assigned select options when adding predunning**
Adding a new option to a select field via API replaced ALL existing options instead of appending. All sequence values became invalid, causing 400 errors on contact sync.
**Rule: When modifying Mautic select field options via API, always GET the current options first, append the new one, then PUT the full list back.**

**Overall prevention**: Mautic's API is unreliable for campaign management. Always verify state after every API call. Use the DB directly for reads (`mysql -u mautic -pmautic mautic`). Treat every Mautic operation as potentially state-corrupting and verify independently.

---

### 2026-04-21 — Bulk-sent 1333 emails at ~3/sec without stating rate, tanked deliverability
**What happened**: Ran a paddle-migration pilot blast with `time.sleep(0.3)` between API sends. ~3 emails/sec sustained. Mautic + SES handled it, but Gmail's inbound rate-limiter flagged it as suspicious bulk — emails landed in Promotions/Spam rather than Primary. Combined with cold path (first ever mass send from seedr.cc) = open rate lower than it should have been. Worse: I never even stated the rate before firing. User had to ask + check the script to discover what had happened.
**Root cause**: Treated "API succeeds" as "delivery is fine". Didn't state send parameters (rate, batch size, ramp strategy) before user consented. Gmail-side ratelimit and reputation warmup got zero thought.
**Prevention rule**: **NEVER fire a bulk send without stating the parameters out loud FIRST.** For every blast >100 recipients, print:
- Total target count
- Sends per second / per minute
- Total estimated duration
- From-domain reputation (is this a cold path? new template?)
- Any warmup ramp plan

User must explicitly confirm. Defaults that are actually safe:
- **Domain not warmed / cold subject line**: ≤ 100/hr day 1, ≤ 500/hr day 2, ramp to 1000/hr week 1, up from there
- **Warmed domain, transactional-looking subject**: 1-2/sec (60-120/min)
- **NEVER 3+/sec unless SES + Gmail have been warmed with this sender+template style already**

---

### 2026-04-20/21 — Claimed click tracking worked based on URL shape, actually tracked nothing
**What happened**: Paddle pilot blasted 1333 emails. CTA used multi-line `<a>` tag + `{contactfield=X}` tokens inside the URL. Mautic's URL-wrap regex silently skipped the CTA → no `/r/HASH` in delivered body → zero clicks recorded in `page_hits`. I spent hours claiming tracking worked because (a) SES awstrack.me wrap was present, (b) Matomo showed some numbers, (c) API responses said success. Actual first-click capture rate on real users: effectively 0.
**Root cause**: Never verified end-to-end. Trusted URL structure and API responses instead of querying `page_hits` after a real click from a real recipient.
**Prevention rule**: See `emails.md` section "Click tracking — source of truth" and "Before bulk-send (pre-flight)". Key: `page_hits` is the ONLY reliable click counter. SES awstrack.me events are mostly Gmail Safe Browsing bots (172.253.x.x, 66.249.x.x IP ranges). Matomo is blocked by ~25-50% of real user browsers (Brave, uBlock, Firefox ETP). **Always fire a test send, click from a clean browser, `SELECT COUNT(*) FROM page_hits WHERE lead_id=<test> AND email_id=<id> AND date_hit>=NOW()-INTERVAL 1 MINUTE` before believing any tracking claim.**

---

### 2026-04-20 — Mautic email_stats table 74GB, indexes corrupted mid-blast
**What happened**: 1333-row bulk blast hammered email_stats. 14 secondary indexes became corrupt (InnoDB). Mautic UI went down (500s on every page reading the table). Open tracking silently failed for ~90 min.
**Root cause**: Table had grown unchecked for years (no cleanup cron). Under concurrent INSERT+UPDATE load, latent fragility blew up.
**Prevention rule**: See `emails.md` section "Bulk blast — DB health". Before any blast of >500 recipients:
1. `ls -lah /var/lib/mysql/mautic/email_stats.ibd` — if >50GB, rebuild first
2. `CHECK TABLE email_stats` — if warnings, rebuild before sending
3. Schedule `mautic:emails:stats:cleanup` monthly via cron

Rebuild is ZERO-downtime with `ALGORITHM=INPLACE, LOCK=NONE` (see `emails.md` for exact SQL).

---

### 2026-05-11 — Android inner-shadow vignette from overflow+borderRadius+elevation combo
**What happened**: Focused/active states on TV (video tiles, back button, settings gear, Incoming card) showed a dark "inner shadow" / vignette band just inside the focus border. Looked horrible. Three rounds of agent dispatches kept reintroducing it because each agent inlined its own shadow block.
**Root cause**: On Android, when a single `<View>` has ALL THREE of:
- `overflow: 'hidden'`
- non-zero `borderRadius`
- non-zero `elevation`

…the system clips the elevation shadow inward, producing a dark inset band at the rounded inner edge. This is a known Android RN/Yoga rendering quirk. iOS doesn't have it (iOS uses `shadow*` props which don't clip).

**Prevention rule (MANDATORY for all agents touching focus/active visuals)**:
- **NEVER combine `overflow: 'hidden'` + `borderRadius` + `elevation` on the same Android `<View>`.**
- For TV focus indication, prefer: border color/width change + scale transform (`transform: [{ scale: 1.02 }]`). NO `elevation` on the clipped wrapper.
- iOS shadow props (`shadowColor`/`shadowOffset`/`shadowOpacity`/`shadowRadius`) ARE safe to keep — iOS doesn't clip them. Drop only `elevation`.
- The shared `shadow.shadowFocusGlow` token in `android-plugin/src/theme/mockupTokens.js` has `elevation` intentionally removed (commit `b6bd6f9b`). Use the token — DO NOT inline custom shadow blocks that re-add elevation.
- If a focus visual genuinely needs Android elevation, restructure: outer wrapper with elevation (no overflow:hidden, no border-radius) wrapping inner clipped child. Two views, not one.

**Detection**: Screenshot of the focused state on an Android emulator. If you see a dark band BETWEEN the border and the content, you've hit the bug.

**Cleanup commits**: `b6bd6f9b` (strip elevation from `shadowFocusGlow` token), `cfaee69f` (migrate VideoCard to MediaTile), and the IncomingView fix on `2026-05-11`.
