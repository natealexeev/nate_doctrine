# Nate's Doctrine

> Universal principles for all software projects.

## Documents

| Document | Scope |
|----------|-------|
| [frontend.md](frontend.md) | Frontend architecture, components, stores, testing |
| backend.md | *(coming soon)* |
| testing.md | *(coming soon)* |
| refactoring.md | *(coming soon)* |

## The Supreme Laws

### 1. ALL REAL LOGIC BELONGS IN THE BACKEND

Frontend is a thin integration layer. It calls endpoints and displays results. Business logic, calculations, validations, AI calls - all backend. Violation is **HOCHVERRAT**.

### 2. THE CODE MUST BE SIMPLE

- Extract functionality to separate classes
- Preserve simple structures and stateless flows
- UI is disposable
- Simple one-liners stay inline
- Don't over-engineer

### 3. TESTS PROVE FUNCTIONALITY, NOT EXISTENCE

Every test must answer:
- If this passes, what is proven true?
- If this fails, what broke?

No theatre tests. No mocking things to prove they match mocks.

### 4. ALTERNATIVE ACCESS PIPELINES

Everything accessible via:
- UI (user clicks)
- Direct API (curl, scripts)
- AI agents (automation)
- Tests (verification)

### 5. NEVER SWALLOW ERRORS

All catch blocks must log and surface errors. No empty catches.

---

*See individual doctrine files for detailed rules and examples.*
