# The Testing Doctrine

> Principles governing all testing decisions across all projects.

---

## The Supreme Law

### TESTS PROVE FUNCTIONALITY, NOT EXISTENCE

Every test must answer two questions:
- **If this passes, what is proven true?**
- **If this fails, what broke?**

If you can't answer clearly, you're writing theatre.

---

## What to Test

### Test This

**Pure logic functions** - input -> output, edge cases matter:
```typescript
describe('detectAnomalies', () => {
  it('returns empty for insufficient data', () => {
    expect(detectAnomalies([1, 2, 3])).toEqual([]);
  });

  it('detects statistical outliers', () => {
    const values = [10, 11, 9, 10, 100]; // 100 is outlier
    expect(detectAnomalies(values)).toContain(4);
  });
});
```

**Calculations, validations, transformations:**
- Math operations with edge cases (division by zero, negative numbers)
- Data transformations (sorting, filtering, grouping)
- Business rules and conditional logic
- Format conversions

**Schema validation at load time:**
```typescript
// Fixtures validate against schemas when module loads
// Wrong fixture = immediate failure, no test needed
export const mockUser = UserSchema.parse({
  id: 1,
  email: 'test@example.com',
  active: true
});
```

---

## What NOT to Test (Theatre)

### API Clients with Mocked Responses

```typescript
// THEATRE - proves nothing
mock.onGet('/users').reply(200, mockUsers);
const result = await usersApi.list();
expect(result.data).toHaveLength(2);
// We defined the mock, we defined the code
// This proves... they match? So what?
```

You're testing that your mock returns what you told it to return. This proves nothing about whether the real API works or whether your code handles real responses correctly.

### Browser API Wrappers

Testing localStorage wrappers, sessionStorage helpers, etc. requires mocking the browser. You end up testing that your mock behaves like you told it to.

### Config/Dictionary Lookups

```typescript
// THEATRE - duplicates the config
expect(getConfig('apiUrl')).toBe('https://api.example.com');
```

The test just duplicates what's in the config file. If the config changes, you update both places. Pointless.

### Trivial Stdlib Wrappers

Don't test thin wrappers around standard library functions. Trust the stdlib.

---

## Theatre vs Proof

| Theatre | Proof |
|---------|-------|
| Tests mock choreography | Tests real logic |
| Proves code matches mock | Proves functionality works |
| Fails when mock changes | Fails when logic breaks |
| Requires extensive setup | Minimal dependencies |
| Tests implementation | Tests behavior |

**The mock test trap:**
```typescript
// You write this
mock.onPost('/users').reply(201, { id: 1 });
const result = await createUser({ name: 'Test' });
expect(result.id).toBe(1);

// What did you prove?
// - That axios-mock-adapter works? Yes.
// - That your API client works with real API? No.
// - That the backend creates users correctly? No.
```

---

## Test Safety

### Isolation Rules

```typescript
// tests/setup.ts
export const mock = new MockAdapter(apiClient, {
  onNoMatch: 'throwException'  // Fail loudly if unmocked endpoint called
});

beforeEach(() => {
  mock.reset();
});
```

**Hard rules:**
1. NEVER create your own HTTP client instance - use the shared client
2. NEVER use native fetch() - use the project's HTTP client only
3. Watch for third-party deps that make HTTP calls - mock or stub them

Violations = tests hitting real APIs = data corruption = gruesome death.

### Environment Isolation

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    // DO NOT load .env - tests must be isolated from real APIs
    envFile: false,
  }
});
```

Tests should never accidentally hit production. The mock adapter intercepts everything, but defense in depth matters.

---

## Fixture Validation

Fixtures validate against schemas at module load time. This is NOT a test - it's a compile-time check.

```typescript
// tests/fixtures/users.ts
import { UserSchema } from '@/schemas/user';

// Schema.parse() validates at module load
// Wrong data = crash before any test runs
export const mockUsers = [
  UserSchema.parse({
    id: 84521,
    email: 'test@example.com',
    active: true,
  }),
];
```

**Benefits:**
- If schema changes and fixture doesn't match -> Zod throws -> immediate failure
- No separate test needed to verify fixture structure
- Single source of truth (schema) governs all

---

## Test File Structure

```
tests/
├── setup.ts               # Mock adapter config, global setup
├── fixtures/              # Mock data (validates against schemas)
│   ├── users.ts
│   ├── orders.ts
│   └── index.ts
└── operations/            # Pure logic tests ONLY
    ├── statistics.test.ts
    ├── validation.test.ts
    └── formatting.test.ts
```

**Note:** No `tests/api/` directory. API client tests are theatre.

---

## The Litmus Test

Before writing a test, ask:

1. **What does this prove?**
   - If you can't articulate it in one sentence, don't write it

2. **Could this fail for the wrong reason?**
   - If the test fails because a mock changed, it's theatre

3. **Does this test real logic?**
   - Inputs, outputs, edge cases, error conditions

4. **Am I testing my code or my mocks?**
   - If you're asserting mock behavior, stop

5. **If this passes, am I confident the feature works?**
   - If no, the test is incomplete or testing the wrong thing

---

## Summary

```
TEST:     Pure logic, calculations, transformations, edge cases
DON'T:    API clients, browser wrappers, config lookups, mocks

FIXTURES: Validate against schemas at load time (not in tests)
SAFETY:   Mock adapter catches unmocked calls, env isolation

QUESTION: If this passes, what is proven true?
          If this fails, what broke?
```

**Remember:** A test that can't fail for a meaningful reason is theatre. Delete it.
