---
description: Conduct a five-axis code review — correctness, readability, architecture, security, performance
---

Invoke the agent-skills:code-review-and-quality skill.

Review the current changes (staged or recent commits) across all five axes:

1. **Correctness** — Does it match the spec? Edge cases handled? Tests adequate?
2. **Readability** — Clear names? Straightforward logic? Well-organized?
3. **Architecture** — Follows existing patterns? Clean boundaries? Right abstraction level?
4. **Security** — Input validated? Secrets safe? Auth checked? (Use agent-skills:security-and-hardening)
5. **Performance** — No N+1 queries? No unbounded ops? (Use agent-skills:performance-optimization)

For changes in the StackRox UI codebase, also load:

- agent-skills:stackrox-ui-conventions — file placement, naming, import order, directory boundaries
- agent-skills:patternfly-development — PatternFly 6 component selection, deprecated APIs, a11y patterns
- agent-skills:rest-and-service-layer — REST data layer conformance (services, hooks, error handling)
- agent-skills:graphql-and-data-layer — only when the diff touches existing GraphQL or backend-authored endpoints
- agent-skills:react-state-patterns — local vs Redux/saga state choices

Use the `agents/frontend-reviewer.md` persona for the React/TypeScript/PatternFly six-dimension pass on UI changes.

Categorize findings as Critical, Important, or Suggestion.
Output a structured review with specific file:line references and fix recommendations.
