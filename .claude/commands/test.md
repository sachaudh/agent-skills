---
description: Run TDD workflow — write failing tests, implement, verify. For bugs, use the Prove-It pattern.
---

Invoke the agent-skills:test-driven-development skill.

For new features:
1. Write tests that describe the expected behavior (they should FAIL)
2. Implement the code to make them pass
3. Refactor while keeping tests green

For bug fixes (Prove-It pattern):
1. Write a test that reproduces the bug (must FAIL)
2. Confirm the test fails
3. Implement the fix
4. Confirm the test passes
5. Run the full test suite for regressions

For browser-related issues, also invoke agent-skills:browser-testing-with-devtools to verify with Chrome DevTools MCP.

For end-to-end coverage of a user flow in the StackRox UI codebase, invoke agent-skills:cypress-e2e-testing and write the spec under `ui/apps/platform/cypress/integration/`. Pair component/unit tests (Vitest + @testing-library/react) with at least one Cypress flow when adding a new page, route, or wizard.
