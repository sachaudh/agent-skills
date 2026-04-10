# Getting Started with StackRox Frontend Agent Skills

This repository is a portable AI brain for StackRox frontend engineering. It contains skills, agent personas, references, and slash commands tuned for React 18.2, TypeScript 5.9, PatternFly 6, Apollo Client 3.8, Redux 4, and the StackRox UI codebase.

It works with any AI coding agent that accepts Markdown instructions (Claude Code, Cursor, Copilot, Gemini CLI, etc.). This guide covers the universal approach.

## How Skills Work

Each skill is a Markdown file (`SKILL.md`) that describes a specific engineering workflow. When loaded into an agent's context, the agent follows the workflow -- including verification steps, anti-patterns to avoid, and exit criteria.

**Skills are not reference docs.** They are step-by-step processes the agent follows.

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/sachaudh/agent-skills.git
```

### 2. Choose a skill

Browse the `skills/` directory. Each subdirectory contains a `SKILL.md` with:
- **When to use** -- triggers that indicate this skill applies
- **Core Process** -- step-by-step workflow
- **Verification** -- how to confirm the work is done
- **Common Rationalizations** -- excuses the agent might use to skip steps
- **Red Flags** -- signs the skill is being violated

### 3. Load the skill into your agent

Copy the relevant `SKILL.md` content into your agent's system prompt, rules file, or conversation. The most common approaches:

**System prompt:** Paste the skill content at the start of the session.

**Rules file:** Add skill content to your project's rules file (CLAUDE.md, .cursorrules, etc.).

**Conversation:** Reference the skill when giving instructions: "Follow the test-driven-development process for this change."

### 4. Use the meta-skill for discovery

Start with the `using-agent-skills` skill loaded. It contains a flowchart that maps task types to the appropriate skill.

## Recommended Setup

### Minimal (Start here)

Load four essential skills into your rules file:

1. **stackrox-ui-conventions** -- Directory structure, naming, and feature organization
2. **spec-driven-development** -- For defining what to build
3. **test-driven-development** -- For proving it works
4. **code-review-and-quality** -- For verifying quality before merge

These four cover the most critical quality gaps for StackRox frontend work.

### Full Lifecycle

For comprehensive coverage, load skills by phase:

```
Starting a feature:  spec-driven-development → planning-and-task-breakdown
During development:  incremental-implementation + test-driven-development
                     + stackrox-ui-conventions (always)
Before merge:        code-review-and-quality + security-and-hardening
Before deploy:       shipping-and-launch
```

## Context-Aware Loading by Task Type

Do not load all skills at once -- it wastes context. Load skills that match the task:

| Task | Load these skills |
|------|-------------------|
| Building a new PatternFly component | `patternfly-development` + `stackrox-ui-conventions` + `frontend-ui-engineering` |
| Fetching or mutating REST data for a new feature | `rest-and-service-layer` + `stackrox-ui-conventions` |
| Modifying an existing GraphQL query or mutation | `graphql-and-data-layer` + `stackrox-ui-conventions` |
| Wiring Redux/saga state | `react-state-patterns` + `stackrox-ui-conventions` |
| Writing a Cypress E2E test | `cypress-e2e-testing` + `test-driven-development` |
| Writing a Vitest unit/component test | `test-driven-development` |
| Migrating to a new PatternFly major version | `deprecation-and-migration` + `patternfly-development` |
| Debugging a broken render or effect loop | `debugging-and-error-recovery` + `browser-testing-with-devtools` |
| Hardening a form or auth flow | `security-and-hardening` + `references/security-checklist.md` |
| Reviewing a PR | `code-review-and-quality` + `agents/frontend-reviewer.md` |
| Onboarding to the StackRox UI codebase | `agents/stackrox-ui-guide.md` + `stackrox-ui-conventions` |

## Skill Anatomy

Every skill follows the same structure:

```
YAML frontmatter (name, description)
├── Overview -- What this skill does
├── When to Use -- Triggers and conditions
├── Core Process -- Step-by-step workflow
├── Examples -- Code samples and patterns
├── Common Rationalizations -- Excuses and rebuttals
├── Red Flags -- Signs the skill is being violated
└── Verification -- Exit criteria checklist
```

See [skill-anatomy.md](skill-anatomy.md) for the full specification.

## StackRox-Specific Content

### New Skills

| Skill | Purpose |
|-------|---------|
| `skills/stackrox-ui-conventions/` | Directory structure, naming, feature organization, component composition rules |
| `skills/patternfly-development/` | PatternFly 6 component selection, theming, accessibility, wrap-vs-direct decisions |
| `skills/rest-and-service-layer/` | REST-first data fetching via `services/` + `useRestQuery` / `useRestMutation` / `usePaginatedQuery`, cancellation, search, error formatting |
| `skills/graphql-and-data-layer/` | Apollo Client 3.8 query/mutation patterns for existing GraphQL code and new backend-authored GraphQL endpoints |
| `skills/react-state-patterns/` | Apollo cache, Redux + saga, Context, and local state decision tree |
| `skills/cypress-e2e-testing/` | Page objects, `cy.intercept()` for GraphQL, `cy.session()` auth, PF testing |

### Customized Skills

| Skill | What changed |
|-------|--------------|
| `skills/context-engineering/` | Added StackRox UI context loading (PatternFly docs, Apollo queries, directory map, Redux/saga) |
| `skills/security-and-hardening/` | Reframed around React XSS, Formik/Yup, CSP, auth token handling |
| `skills/deprecation-and-migration/` | Added PatternFly upgrades, React Router v5-to-v6, redux-form-to-Formik, codemods |

## Using Agents

The `agents/` directory contains pre-configured agent personas:

| Agent | Purpose |
|-------|---------|
| `code-reviewer.md` | Generic five-axis code review |
| `frontend-reviewer.md` | React/TypeScript/PatternFly six-dimension reviewer |
| `stackrox-ui-guide.md` | Onboarding persona -- explains directory structure and data flow |
| `security-auditor.md` | Vulnerability detection |
| `test-engineer.md` | Test strategy and writing |

Load an agent definition when you need specialized review. For example, ask your coding agent to "review this change using the frontend-reviewer agent persona" and provide the agent definition.

## Using Commands

The `.claude/commands/` directory contains slash commands for Claude Code:

| Command | Skill Invoked |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/code-simplify` | code-simplification |
| `/ship` | shipping-and-launch |

## Using References

The `references/` directory contains supplementary checklists:

| Reference | Use With |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening (frontend-focused) |
| `accessibility-checklist.md` | frontend-ui-engineering, patternfly-development |
| `patternfly-checklist.md` | patternfly-development |
| `stackrox-ui-patterns.md` | stackrox-ui-conventions (table pages, detail panels, wizards, filter bars, empty states, error boundaries) |

Load a reference when you need detailed patterns beyond what the skill covers.

## Spec and Task Artifacts

The `/spec` and `/plan` commands create working artifacts (`SPEC.md`, `tasks/plan.md`, `tasks/todo.md`). Treat them as **living documents** while the work is in progress:

- Keep them in version control during development so the human and the agent have a shared source of truth.
- Update them when scope or decisions change.
- If your repo does not want these files long-term, delete them before merge or add the folder to `.gitignore` -- the workflow does not require them to be permanent.

## Tips

1. **Always load `stackrox-ui-conventions`** when touching the StackRox UI -- it is the foundation other StackRox skills reference.
2. **Start with spec-driven-development** for any non-trivial work.
3. **Always load test-driven-development** when writing code.
4. **Do not skip verification steps** -- they are the whole point.
5. **Load skills selectively** -- more context is not always better.
6. **Use the agents for review** -- different perspectives catch different issues.
