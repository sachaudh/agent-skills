---
description: Run the pre-launch checklist and prepare for production deployment
---

Invoke the agent-skills:shipping-and-launch skill.

Run through the complete pre-launch checklist:

1. **Code Quality** — Tests pass, build clean, lint clean, no TODOs, no console.logs
2. **Security** — npm audit clean, no secrets in code, auth in place, headers configured
3. **Performance** — Core Web Vitals good, no N+1 queries, images optimized, bundle sized
4. **Accessibility** — Keyboard nav works, screen reader compatible, contrast adequate
5. **Infrastructure** — Env vars set, migrations ready, monitoring configured
6. **Documentation** — README current, ADRs written, changelog updated

For StackRox UI changes, also load:

- agent-skills:stackrox-ui-conventions — confirm directory placement and naming match conventions
- agent-skills:patternfly-development plus `references/patternfly-checklist.md` — verify PF6 component usage and a11y patterns
- `references/accessibility-checklist.md` — full keyboard, focus, screen reader, contrast pass
- agent-skills:cypress-e2e-testing — verify the relevant Cypress flows are green

Report any failing checks and help resolve them before deployment.
Define the rollback plan before proceeding.
