---
name: cypress-e2e-testing
description: Writes Cypress 15 end-to-end tests for StackRox UI. Use when creating E2E tests, intercepting GraphQL requests, handling auth in tests, testing PatternFly components, or configuring Cypress for CI integration.
---

# Cypress E2E Testing

## Overview

StackRox UI uses Cypress 15 for end-to-end testing. E2E tests verify that features work from the user's perspective -- clicking buttons, filling forms, navigating pages, and seeing correct data. This skill covers page object patterns, GraphQL interception, auth session handling, PatternFly component testing, and CI pipeline integration.

## When to Use

- Writing a new E2E test for a feature
- Testing a user workflow end-to-end
- Intercepting GraphQL requests for deterministic test data
- Setting up authentication in test suites
- Testing PatternFly components (selects, modals, tables)
- Debugging flaky Cypress tests
- NOT for unit or component tests (see `test-driven-development`)
- NOT for browser DevTools debugging (see `browser-testing-with-devtools`)

## Page Object Pattern

Encapsulate page interactions in page objects to keep tests readable and reduce duplication:

```tsx
// cypress/support/pages/VulnerabilitiesPage.ts

export class VulnerabilitiesPage {
  static url = '/main/vulnerabilities';

  static visit() {
    cy.visit(this.url);
    cy.get('[data-ouia-component-id="vulnerabilities-page"]').should('exist');
  }

  static getTable() {
    return cy.get('[data-ouia-component-id="vulnerabilities-table"]');
  }

  static getTableRows() {
    return this.getTable().find('tbody tr');
  }

  static searchFor(term: string) {
    cy.get('[data-ouia-component-id="vulnerability-search"]')
      .clear()
      .type(term);
  }

  static selectSeverity(severity: string) {
    cy.get('[data-ouia-component-id="severity-filter"]').click();
    cy.get(`[data-ouia-component-id="severity-option-${severity}"]`).click();
  }

  static getRowByCVE(cve: string) {
    return this.getTable().contains('tr', cve);
  }

  static clickRowAction(cve: string, actionLabel: string) {
    this.getRowByCVE(cve).within(() => {
      cy.get('[data-ouia-component-id="row-actions"]').click();
    });
    cy.get('[role="menuitem"]').contains(actionLabel).click();
  }
}
```

### Using Page Objects in Tests

```tsx
// cypress/e2e/vulnerabilities.cy.ts
import { VulnerabilitiesPage } from '../support/pages/VulnerabilitiesPage';

describe('Vulnerabilities', () => {
  beforeEach(() => {
    cy.loginAsAdmin();
    VulnerabilitiesPage.visit();
  });

  it('displays vulnerability results', () => {
    VulnerabilitiesPage.getTableRows().should('have.length.greaterThan', 0);
  });

  it('filters by severity', () => {
    VulnerabilitiesPage.selectSeverity('CRITICAL');
    VulnerabilitiesPage.getTableRows().each(($row) => {
      cy.wrap($row).should('contain.text', 'Critical');
    });
  });

  it('searches by CVE ID', () => {
    VulnerabilitiesPage.searchFor('CVE-2024-1234');
    VulnerabilitiesPage.getTableRows().should('have.length', 1);
    VulnerabilitiesPage.getRowByCVE('CVE-2024-1234').should('exist');
  });
});
```

## GraphQL Interception with `cy.intercept()`

Intercept GraphQL requests to control test data and avoid flaky tests caused by backend state:

### Intercepting by Operation Name

```tsx
// Intercept a specific GraphQL operation
cy.intercept('POST', '/api/graphql', (req) => {
  if (req.body.operationName === 'GetVulnerabilities') {
    req.reply({
      data: {
        vulnerabilities: [
          {
            id: '1',
            cve: 'CVE-2024-1234',
            severity: 'CRITICAL',
            fixable: true,
            component: 'openssl',
            version: '1.1.1k',
          },
        ],
      },
    });
  }
});
```

### Reusable GraphQL Intercept Helper

```tsx
// cypress/support/graphql.ts

/**
 * Intercept a GraphQL operation by name and return mock data.
 */
export function mockGraphQL(
  operationName: string,
  responseData: Record<string, unknown>,
  alias?: string
) {
  const interceptAlias = alias ?? operationName;
  cy.intercept('POST', '/api/graphql', (req) => {
    if (req.body.operationName === operationName) {
      req.reply({ data: responseData });
    }
  }).as(interceptAlias);
}

/**
 * Wait for a GraphQL operation to complete.
 */
export function waitForGraphQL(operationName: string) {
  cy.wait(`@${operationName}`);
}

/**
 * Intercept a GraphQL operation and let it pass through to the server,
 * but alias it for waiting.
 */
export function spyOnGraphQL(operationName: string) {
  cy.intercept('POST', '/api/graphql', (req) => {
    if (req.body.operationName === operationName) {
      req.alias = operationName;
    }
  });
}
```

### Using in Tests

```tsx
import { mockGraphQL, waitForGraphQL } from '../support/graphql';

describe('Vulnerability Details', () => {
  it('shows vulnerability detail panel', () => {
    mockGraphQL('GetVulnerabilityDetail', {
      vulnerability: {
        id: '1',
        cve: 'CVE-2024-1234',
        severity: 'CRITICAL',
        description: 'Buffer overflow in OpenSSL',
        affectedDeployments: 5,
      },
    });

    VulnerabilitiesPage.visit();
    VulnerabilitiesPage.getRowByCVE('CVE-2024-1234').click();

    waitForGraphQL('GetVulnerabilityDetail');

    cy.get('[data-ouia-component-id="vulnerability-detail-panel"]')
      .should('contain.text', 'Buffer overflow in OpenSSL')
      .and('contain.text', '5 affected deployments');
  });
});
```

## Authentication with `cy.session()`

Use `cy.session()` to cache auth state across tests and avoid re-logging in:

```tsx
// cypress/support/commands.ts

Cypress.Commands.add('loginAsAdmin', () => {
  cy.session('admin', () => {
    cy.request({
      method: 'POST',
      url: '/v1/auth/login',
      body: {
        username: Cypress.env('ADMIN_USERNAME'),
        password: Cypress.env('ADMIN_PASSWORD'),
      },
    }).then((response) => {
      window.localStorage.setItem('access_token', response.body.token);
    });
  }, {
    validate() {
      cy.request({
        url: '/v1/auth/status',
        failOnStatusCode: false,
      }).its('status').should('eq', 200);
    },
  });
});

Cypress.Commands.add('loginAsReadOnly', () => {
  cy.session('readonly', () => {
    cy.request({
      method: 'POST',
      url: '/v1/auth/login',
      body: {
        username: Cypress.env('READONLY_USERNAME'),
        password: Cypress.env('READONLY_PASSWORD'),
      },
    }).then((response) => {
      window.localStorage.setItem('access_token', response.body.token);
    });
  });
});
```

### Auth Credentials

Store credentials in `cypress.env.json` (gitignored) or CI environment variables:

```json
{
  "ADMIN_USERNAME": "admin",
  "ADMIN_PASSWORD": "admin-password",
  "READONLY_USERNAME": "readonly",
  "READONLY_PASSWORD": "readonly-password"
}
```

## Testing PatternFly Components

PatternFly components use `data-ouia-component-id` and `data-ouia-component-type` attributes for test hooks.

### Selecting PatternFly Components

```tsx
// Select by OUIA ID (preferred -- stable, explicit)
cy.get('[data-ouia-component-id="save-policy-btn"]').click();

// Select by OUIA component type (when ID is not set)
cy.get('[data-ouia-component-type="PF6/Button"]').contains('Save').click();

// PatternFly Table rows
cy.get('[data-ouia-component-type="PF6/TableRow"]').should('have.length', 5);
```

### Common PatternFly Testing Patterns

```tsx
// Select component (PF6 dropdown-based)
function selectOption(ouiaId: string, optionText: string) {
  cy.get(`[data-ouia-component-id="${ouiaId}"]`).click();
  cy.get('[role="option"]').contains(optionText).click();
}

// Modal dialog
function confirmModal() {
  cy.get('[data-ouia-component-type="PF6/ModalContent"]').within(() => {
    cy.get('button').contains('Confirm').click();
  });
}

// Alert banner
function assertAlert(variant: string, message: string) {
  cy.get(`[data-ouia-component-type="PF6/Alert"][class*="${variant}"]`)
    .should('contain.text', message);
}

// Tabs
function switchTab(tabLabel: string) {
  cy.get('[data-ouia-component-type="PF6/TabButton"]')
    .contains(tabLabel)
    .click();
}

// Pagination
function goToPage(pageNumber: number) {
  cy.get('[data-ouia-component-type="PF6/Pagination"]').within(() => {
    cy.get('input[aria-label="Current page"]')
      .clear()
      .type(`${pageNumber}{enter}`);
  });
}
```

## CI Integration

### Cypress Configuration for CI

```tsx
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: process.env.CYPRESS_BASE_URL || 'https://localhost:8443',
    viewportWidth: 1440,
    viewportHeight: 900,
    defaultCommandTimeout: 10000,
    requestTimeout: 15000,
    retries: {
      runMode: 2,    // Retry twice in CI
      openMode: 0,   // No retries in local dev
    },
    video: true,
    screenshotOnRunFailure: true,
    experimentalMemoryManagement: true,
  },
});
```

### Running in CI

```bash
# Run all E2E tests
npx cypress run --browser chrome

# Run a specific spec
npx cypress run --spec "cypress/e2e/vulnerabilities.cy.ts"

# Run with a specific base URL
CYPRESS_BASE_URL=https://staging.example.com npx cypress run
```

### Handling Flaky Tests

1. **Use `cy.intercept()` for deterministic data** instead of relying on backend state
2. **Use explicit waits** (`cy.wait('@alias')`) instead of arbitrary timeouts
3. **Use `cy.session()` for auth** to avoid login race conditions
4. **Use `.should()` assertions** which auto-retry instead of `.then()` chains
5. **Isolate tests** -- each test should set up its own state, not depend on prior tests

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll test this manually -- E2E tests are slow" | Manual testing doesn't prevent regressions. E2E tests catch integration bugs that unit tests miss. |
| "cy.wait(3000) is good enough" | Arbitrary waits cause flaky tests. Use `cy.wait('@alias')` for network calls and `.should()` for DOM assertions. |
| "I don't need page objects for a small test" | Page objects pay off immediately when the UI changes. One update in the page object fixes all tests. |
| "Mocking GraphQL makes the test unrealistic" | Mocking gives you deterministic, fast tests. Use a small number of integration tests (no mocks) for realism. |
| "Auth setup in every test is too slow" | `cy.session()` caches auth state. After the first test, login takes milliseconds. |

## Red Flags

- `cy.wait(N)` with a numeric timeout instead of an alias
- Test selectors using CSS classes or DOM structure instead of `data-ouia-component-id`
- Tests that depend on execution order (test B assumes test A ran first)
- Login via the UI instead of `cy.session()` with API-based auth
- Inline GraphQL mock data duplicated across multiple tests (extract to fixtures)
- No `cy.intercept()` for GraphQL in tests that assert on specific data
- Missing `beforeEach` cleanup or state reset

## Verification

After writing Cypress E2E tests:

- [ ] Page objects used for page-level interactions
- [ ] Selectors use `data-ouia-component-id` or `data-ouia-component-type`, not CSS classes
- [ ] GraphQL requests intercepted with `cy.intercept()` for deterministic data
- [ ] Auth handled with `cy.session()`, not UI-based login per test
- [ ] No `cy.wait(N)` with numeric timeouts (use aliases or assertions)
- [ ] Tests are independent -- no execution order dependencies
- [ ] Test runs in CI with retries enabled (`retries.runMode >= 1`)
- [ ] Video and screenshots enabled for CI failure debugging
- [ ] Mock data extracted to fixtures or helper functions when reused
