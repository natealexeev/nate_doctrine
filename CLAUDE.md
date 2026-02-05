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

### 2. DUPLICATION IS DIE HÖCHSTE VERRAT

No component created without searching for existing solutions first. No refactoring that introduces duplication. If something does 80% of what you need, USE IT.

Before creating ANY component:
- Does this already exist?
- Can an existing component be extended?
- Is this truly reusable or a one-off?

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

*See individual doctrine files for detailed rules and examples.*
