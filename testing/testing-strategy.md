# Testing Strategy

The testing approach covers unit tests, integration tests, component tests, and end-to-end logic tests. Tests are organized by domain to make them easy to maintain and extend. This document covers the full testing philosophy, patterns, infrastructure, and quality metrics.

---

## What You Will Learn

- How tests are organized and what each test type covers
- How the test pyramid is applied to this project
- How mocking strategies work for different dependencies
- How snapshot testing catches UI regressions
- How tests integrate with the CI/CD pipeline
- What coverage targets are enforced and how they are measured

---

## Testing Philosophy

The project follows these testing principles:

1. **Test behavior, not implementation** - Tests verify what the code does, not how it does it. Refactoring should not break tests
2. **Domain-aligned organization** - Tests are organized the same way as source code. Service tests are with services, reducer tests are with reducers
3. **Fast feedback** - Unit tests run in milliseconds. Component tests run in seconds. The full suite completes in under 2 minutes
4. **Deterministic** - Tests do not depend on external services, network access, or time-dependent data. Mocks and fixtures provide predictable inputs
5. **Self-documenting** - Test names describe the behavior being verified. A failing test name tells you what broke without reading the assertion

---

## Test Pyramid

```
        /\
       /  \
      / E2E \
     / ------ \
    /Integration\
   /------------- \
  /   Unit Tests   \
 /-------------------\
```

### Unit Tests (70%)

The largest layer. Test individual functions, hooks, and utilities in complete isolation.

**What is tested:**
- Utility functions (currency formatting, date handling, array sorting)
- Service functions (API calls, data transformation, error handling)
- Custom hooks (state management, side effects)
- Selectors (Redux state derivation)

**What is NOT tested:**
- React component rendering (that is for component tests)
- API integration (that is for integration tests)
- Browser APIs (those are mocked)

### Component Tests (20%)

Test that components render correctly and handle user interactions.

**What is tested:**
- Form validation (required fields, format validation, error messages)
- Loading states (skeleton display, spinner visibility)
- Error states (empty states, error messages, retry buttons)
- User interactions (button clicks, form submission, navigation)
- Conditional rendering (feature flags, permissions-based visibility)

**What is NOT tested:**
- Styling and visual appearance (that is manual QA)
- Animation behavior (too brittle)
- Third-party component internals (test their public API only)

### Integration Tests (10%)

Test that multiple units work together correctly.

**What is tested:**
- Redux reducer chains (dispatching multiple actions)
- Service layer with mocked API responses
- Component + Redux integration
- Form submission flow (validation -> API call -> success/error)

### E2E Tests (Ad Hoc)

Manual smoke tests and critical path validations. No automated E2E framework is currently in use, but critical user flows are documented for manual regression testing.

---

## Test Organization

Tests mirror the source code structure:

```javascript
src/
  services/
    auth.js
    helper.js
    sorting.js
  __tests__/
    services/
      Service_Auth.test.js    // Tests for auth.js
      Service_Global.test.js  // Tests for helper.js
      Data_Sorting.test.js    // Tests for sorting.js
    components/
      LoginPage.test.js
      SignupPage.test.js
    reducers/
      Auth_Reducer.test.js
      Sales_Reducer.test.js
    logic/
      Data_Sync_Logic.test.js
      PPC_Campaign_Flow.test.js
```

### Naming Conventions

| Test Type      | File Pattern                     | Example                                |
| -------------- | -------------------------------- | -------------------------------------- |
| Service tests  | `Service_<Domain>.test.js`       | `Service_Auth.test.js`                 |
| Reducer tests  | `<Slice>_Reducer.test.js`        | `Auth_Reducer.test.js`                 |
| Component tests | `<ComponentName>.test.js`       | `LoginPage.test.js`                    |
| Logic tests    | `<Domain>_<Flow>.test.js`        | `PPC_Campaign_Flow.test.js`            |

---

## Testing Patterns

### 1. Reducer Test Pattern

Reducer tests verify that state transitions produce the expected output:

Tests verify that dispatching `AUTH_USER` correctly populates user data and sets `is_authenticated` to true, while dispatching `LOGOUT` clears all state back to initial values. Each test uses known inputs and expected outputs, making failures easy to diagnose.

**Why this pattern works:** Each test is independent. They do not share state. The `initialState` is defined per test block. Each action is tested with known inputs and expected outputs. This makes failures easy to diagnose.

### 2. Service Test Pattern

Service tests mock the API layer and test data transformation:

Service tests mock the HTTP client at the module level. Each test provides a mock resolved or rejected promise, then verifies that:
1. The API was called with the correct endpoint and payload
2. The response was transformed correctly
3. Errors (like 401) are thrown with the proper message

**Mocking strategy:** `axios` is mocked at the module level using `jest.mock()`. The mock returns a resolved or rejected promise based on the test case. This keeps tests fast (no network calls) and deterministic (no external dependencies).

### 3. Component Test Pattern

Component tests use React Testing Library to render and interact with components:

Component tests use a library that interacts with components the same way a user does — by finding elements through accessible labels, roles, and text. Tests verify that forms show validation errors for empty or invalid inputs by rendering the component with a mock store, simulating user interactions, and checking for expected error messages.

**Why React Testing Library:** Tests interact with the component the same way a user does -- by finding elements by their accessible labels, roles, and text. They do not access component internals, state, or props directly. This means tests break when the behavior changes, not when the implementation changes.

### 4. Logic Test Pattern

Logic tests validate complex business rules that span multiple functions:

Logic tests validate business rules that span multiple functions — for example, verifying that each route maps to the correct sync step, and that the sync status parser handles both legacy array format and new object format correctly.

---

## Mocking Strategy

### What Gets Mocked

| Dependency          | Mock Strategy                                     | Reason                                       |
| ------------------- | ------------------------------------------------- | -------------------------------------------- |
| Axios (API calls)   | `jest.mock('axios')` - mock resolved/rejected     | Prevent real API calls in tests              |
| Firebase            | `__mocks__/firebase.js` - stub functions          | Firebase requires browser APIs not in JSDOM  |
| localStorage        | Wrapped in try-catch or mock with `jest.fn()`     | Not available in test environment            |
| CryptoJS            | `jest.mock('crypto-js')` - return known values    | Encryption is not test-relevant              |
| Chart.js            | Mock the component wrapper                        | Canvas rendering is not testable             |
| Stripe.js           | Mock `loadStripe` and `Elements` wrapper          | Stripe requires browser APIs                 |
| React Router        | Use `MemoryRouter` in component tests             | Control route state in tests                 |

### Mock Implementation Examples

Dependencies that require browser APIs not available in the test environment (like Firebase) are mocked entirely. The mock returns predictable values — `getToken` resolves to a known token, `onMessage` fires a callback with a known payload — keeping tests deterministic and fast.

### Why Axios Is Mocked Instead of Using Mock Service Worker

The project uses `jest.mock('axios')` instead of MSW (Mock Service Worker) because:

1. MSW adds complexity to the test setup
2. Axios mocks are simpler to understand and debug
3. The API layer is thin (just request/response transformation), so mocking at the HTTP level provides little additional value

If the project grows to include more complex API interactions (WebSocket, streaming), MSW would be reconsidered.

---

## Snapshot Testing

Snapshot tests are used sparingly for stable UI components:

Snapshot tests are used sparingly for stable UI components. They capture the rendered output and flag unintended changes on subsequent runs. Snapshots are kept small (individual components, not entire pages) and are checked into version control.

### Snapshot Best Practices

1. **Keep snapshots small** - Test individual components, not entire pages
2. **Review snapshot diffs** - When a snapshot changes, verify the change is intentional before updating
3. **Limit snapshot use** - Prefer explicit assertions over snapshots for behavior verification
4. **Commit snapshots** - Snapshots are checked into version control so CI can detect unexpected changes

---

## CI/CD Integration

### Pipeline Test Execution

Tests run in the CodeBuild pipeline during the `pre_build` phase:

```yaml
pre_build:
    commands:
        - echo Running tests...
        - npm test -- --watchAll=false --passWithNoTests --ci --reporters=default --reporters=jest-junit
        - echo Test coverage report:
        - npm test -- --coverage --watchAll=false --passWithNoTests --ci 2>/dev/null || true
```

### Test Reports

Test results are published to:

- **JUnit XML format** - Integrated with AWS CodeBuild test reporting
- **Coverage HTML report** - Available as a build artifact for review
- **Console output** - Full test output visible in CodeBuild logs

### Failing the Build

The build fails if:

1. Any test suite has a failing test
2. Test coverage drops below configured thresholds (if enforced)
3. Test suite crashes (unhandled exception in test code)

---

## Coverage Metrics

### Current Coverage Areas

| Test Suite                 | Tests | Focus                                        |
| -------------------------- | ----- | -------------------------------------------- |
| Auth tests                 | 15+   | Login, registration, token management        |
| Network tests              | 20+   | API request handling, error responses        |
| Reducer tests              | 40+   | State transitions for all Redux slices       |
| Component tests            | 25+   | Form validation, rendering, interactions     |
| Logic tests                | 18+   | Sync flow, campaign ops, inventory calcs     |
| Utility tests              | 12+   | Formatting, date handling, sorting           |

### Coverage Goals

| Metric           | Current | Target   |
| ---------------- | ------- | -------- |
| Statement coverage | ~60%    | 80%      |
| Branch coverage    | ~50%    | 75%      |
| Function coverage  | ~65%    | 85%      |
| Line coverage      | ~60%    | 80%      |

### Coverage Gaps (Planned Improvements)

| Area                | Gap                                    | Plan                                            |
| ------------------- | -------------------------------------- | ----------------------------------------------- |
| Async operations    | Polling logic, timeout handling        | Add tests for polling service edge cases        |
| Error boundaries    | React error boundary catch blocks      | Add component tests for error fallbacks         |
| Edge cases          | Empty states, null data, network errors | Add logic tests for boundary conditions         |
| Accessibility       | ARIA labels, keyboard navigation       | Add jest-axe for automated a11y testing         |

---

## Interview Talking Points

**On the test philosophy:** "I organize tests by behavior, not implementation. A reducer test verifies that dispatching AUTH_USER transitions the state from logged-out to logged-in. It does not check that the reducer function was called or that internal state variables changed in a specific way. This means refactoring the reducer internals does not break the test. I apply the same principle to component tests -- find elements by accessible labels, not by CSS classes or component state."

**On mocking Firebase:** "Firebase requires browser APIs that are not available in JSDOM. Instead of fighting the test environment, I mock the Firebase module entirely. The mock returns predictable values: `getToken` resolves to a known token, `onMessage` fires a callback with a known payload. This keeps tests deterministic and fast. The Firebase mock lives in `__mocks__/firebase.js` and is automatically applied when tests import from the firebase module."

**On the test pyramid balance:** "Unit tests are 70% of the test suite because they are fast and reliable. Component tests are 20% and cover user interactions. Integration tests are 10% and cover reducer chains and service flows. There are no automated E2E tests because the UI changes frequently and E2E tests are expensive to maintain. Critical user flows are documented for manual regression testing instead."

**On what is hardest to test:** "The polling service was the hardest to test because it involves timers and asynchronous state transitions. I had to use Jest's fake timers to control the polling interval and mock the API responses with different states. The key insight was to make the polling service configurable (interval, max attempts) so tests could run with shorter intervals and fewer attempts."

---

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md) - Component structure
- [State Management](../frontend/state-management.md) - Redux store design
- [Data Flow](../architecture/data-flow.md) - End-to-end data movement
- [CI/CD Pipeline](../deployment/deployment-pipeline.md) - Build and deploy pipeline
- [UI Design System](../frontend/ui-design-system.md) - Styling approach
