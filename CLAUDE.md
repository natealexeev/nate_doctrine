# Nate's Doctrine

> Universal principles for all software projects.

## ⚠️ ALWAYS KEEP THIS REPO SYNCED

**Every edit to this repo MUST be immediately committed and pushed.** No exceptions. Doctrine changes that exist only locally are invisible to other sessions and agents. If you edit a file, you commit and push. Every time.

## Documents

| Document | Scope |
|----------|-------|
| [frontend.md](frontend.md) | Frontend architecture, components, stores, schemas |
| [ux-laws.md](ux-laws.md) | UX heuristics — Jakob's Law, Fitts's Law, Doherty Threshold, etc. |
| [testing.md](testing.md) | Testing philosophy, what to test, theatre vs proof |
| [lessons.md](lessons.md) | **READ FIRST** — Hard-won rules from repeated mistakes. Prevention rules for every recurring error. |
| backend.md | *(coming soon)* |
| refactoring.md | *(coming soon)* |

**Before any expensive ML/compute launch:** read `lessons.md` and apply the launch contract checklist from `2026-06-25 — Launched AG LTV baseline without stating architecture out loud`. State data, labels, included/omitted stages, targets, parallelism, resource risk, ETA, and output/log paths before starting jobs.

## Procedures

| Procedure | Purpose |
|-----------|---------|
| [procedures/browser-testing.md](procedures/browser-testing.md) | Setting up browser testing agents with Chrome DevTools Protocol |
| [procedures/parity-testing.md](procedures/parity-testing.md) | Visual comparison between implementations using ImageMagick |
| `infinite-review-loop` skill | Recursive review-fix cycle until zero issues remain (invoke via Skill tool) |
| [procedures/mockup-procedure.md](procedures/mockup-procedure.md) | Component-based mockups that share design-system.css — no double work |
| [procedures/integrated-mockup.md](procedures/integrated-mockup.md) | Prototype inside the real app — fake data, real components, zero re-integration |

## The Supreme Laws

### 1. ALL REAL LOGIC BELONGS IN THE BACKEND

Frontend is a thin integration layer. It calls endpoints and displays results. Business logic, calculations, validations, AI calls - all backend. Violation is **HOCHVERRAT**.

### 2. DUPLICATION IS DIE HÖCHSTE VERRAT — SUPERSTRICT ZERO TOLERANCE

This applies to **ALL code** — backend, frontend, infrastructure, migrations. Not just components.

**The rule:** If code does the same thing in two places, one of them is wrong. There is no "acceptable duplication." There is no "it's easier to copy-paste." Every duplicate is a future bug where one copy gets updated and the other doesn't.

**Before writing ANY code, you MUST verify:**
- [ ] Does a function/method/handler already do this? → Call it
- [ ] Does a helper/utility already handle this pattern? → Use it
- [ ] Am I about to copy-paste and tweak? → **STOP. Refactor the original to accept parameters**
- [ ] Are two endpoints doing 80% the same query/logic? → Extract shared function
- [ ] Am I reimplementing something the stdlib/framework provides? → Use the built-in

**Backend-specific deduplication:**
- One HTTP client wrapper per external service — never duplicate request/auth logic
- One permission check function — not inline `in_array()` scattered across route files
- One response builder — not ad-hoc `json.Marshal` in handlers
- One config loader — not `yaml.Unmarshal` in multiple places
- Shared SQL query fragments extracted to constants or builder functions
- Middleware registered once, not reimplemented per-route

**The migration principle:** When porting code from one language to another, deduplication is even MORE critical. The original codebase may have grown organically with duplication — the new codebase must NOT inherit it. A migration is an opportunity to consolidate, not to faithfully reproduce every copy-paste.

**The 80% rule:** If existing code does 80% of what you need, extend it. Don't create a "similar but different" version. If you find yourself writing code that looks familiar, STOP and search first.

### 3. THE CODE MUST BE SIMPLE

- Extract functionality to separate classes
- Preserve simple structures and stateless flows
- UI is disposable
- Simple one-liners stay inline
- Don't over-engineer

### 4. TESTS PROVE FUNCTIONALITY, NOT EXISTENCE

Every test must answer:
- If this passes, what is proven true?
- If this fails, what broke?

No theatre tests. No mocking things to prove they match mocks.

### 5. NEVER SWALLOW ERRORS

All catch blocks must log and surface errors. No empty catches.

### 6. UX LAWS ARE NON-NEGOTIABLE FOR ALL FRONTEND WORK

**Every frontend implementation MUST adhere to `ux-laws.md` with EXTREME PRECISION.** This is not a suggestion — it is doctrine.

Before implementing ANY UI component, page, or user flow, you MUST cross-check against all applicable UX laws:
- **Jakob's Law** — Use standard patterns. Don't reinvent navigation.
- **Doherty Threshold** — Feedback within 400ms or use skeleton loaders.
- **Hick's Law** — Minimize choices. Break complex flows into steps.
- **Von Restorff Effect** — Primary CTAs must visually stand out.
- **Fitts's Law** — Interactive targets must be large enough and well-placed.
- **Peak-End Rule** — Success states get delight elements.
- **Tesler's Law** — Shift complexity to the system, not the user.
- **Miller's Law** — Chunk content into groups of 5-9 items.

**Read `nate_doctrine/ux-laws.md` before every frontend task. No exceptions.**

---

## Linked Plans

Active implementation plans that MUST be followed alongside this doctrine:

| Plan | Location | Scope |
|------|----------|-------|
| Golang Conversion | `../seedr-agent-golang/docs/plans/2026-02-09-golang-conversion.md` | PHP → Go backend migration |

## Code Review Policy

**Every code review MUST be evaluated against BOTH:**
1. **This doctrine** (nate_doctrine) — universal principles, deduplication, simplicity, error handling
2. **The relevant plan** (linked above) — specific architectural decisions, endpoint contracts, behavioral changes

A review that only checks the plan but ignores doctrine violations is incomplete. A review that only checks doctrine but ignores plan deviations is incomplete. Both are required.

---

## Merge & Integration Policy

**NEVER merge by direct push, local fast-forward, or silent cherry-pick into a shared branch. EVERY merge goes through a Pull Request, reviewed TWICE before it lands.**

For any branch destined for an integration branch or `main` (feature lanes, fix branches, agent/worktree branches):

1. **Open a PR.** No exceptions — not for "small" changes, not for "obviously safe" ones.
2. **Review the diff yourself** — read the full changeset against this doctrine and the relevant plan.
3. **Review the diff with codex** — run an independent `codex exec` pass. Resolve every BLOCKER before merge.
4. **Merge only after BOTH reviews pass**, then verify the merge added *exactly* the reviewed diff (`git diff --stat` before/after) — nothing else.

Why both reviewers: codex has repeatedly caught real bugs a first pass missed (concurrency cancel-gaps, sign-overflow, follower-cancellation). One reviewer is a blind spot. The PR + post-merge diff check is what prevents silent clobbering (a merge dragging in unintended files).

Note: opening a PR requires pushing the branch — that still needs explicit per-task push approval (push is a "this works" declaration, never automatic).

---

*See individual doctrine files for detailed rules and examples.*
