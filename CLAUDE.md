# Nate's Doctrine

> Universal principles for all software projects.

## Documents

| Document | Scope |
|----------|-------|
| [frontend.md](frontend.md) | Frontend architecture, components, stores, schemas |
| [testing.md](testing.md) | Testing philosophy, what to test, theatre vs proof |
| backend.md | *(coming soon)* |
| refactoring.md | *(coming soon)* |

## Procedures

| Procedure | Purpose |
|-----------|---------|
| [procedures/browser-testing.md](procedures/browser-testing.md) | Setting up browser testing agents with Chrome DevTools Protocol |
| [procedures/parity-testing.md](procedures/parity-testing.md) | Visual comparison between implementations using ImageMagick |

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

*See individual doctrine files for detailed rules and examples.*
