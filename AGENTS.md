# AGENTS.md

This file tells any AI coding agent (Claude Code, Cursor, Copilot, Gemini CLI, OpenCode, etc.) how to use this repository.

## Repository Overview

This is **StackRox Frontend Agent Skills**: a portable AI brain for frontend engineering on the StackRox UI. It contains 27 skills, 5 agent personas, 6 references, and 7 slash commands tuned for React 18.2, TypeScript 5.9, PatternFly 6, Apollo Client 3.8, Redux 4, and the StackRox UI codebase. It is documentation-only (Markdown + YAML frontmatter).

Read `CLAUDE.md` for project identity, conventions, and boundaries. Read `docs/getting-started.md` for how to consume skills.

## Core Rules

- If a task matches a skill, invoke it. Skills encode processes with verification gates -- do not partially apply them.
- Skills live in `skills/<skill-name>/SKILL.md`.
- Follow the skill instructions exactly. Do not skip the Verification section.
- Always load `stackrox-ui-conventions` alongside any other skill when working on StackRox UI code.
- Never duplicate content between skills -- cross-reference instead.
- Never add backend-only content outside clearly labeled "Backend Reference" sections.

## Intent → Skill Mapping

Map user intent to the right skill:

### Generic engineering lifecycle

| Intent | Skill(s) |
|--------|----------|
| Vague idea, needs exploration | `idea-refine` |
| New feature or significant change | `spec-driven-development` → `planning-and-task-breakdown` |
| Implementing code | `incremental-implementation` + `test-driven-development` |
| Writing or running tests | `test-driven-development` |
| Something broke | `debugging-and-error-recovery` |
| Code review | `code-review-and-quality` |
| Refactoring | `code-simplification` |
| API or interface design | `api-and-interface-design` |
| Committing and branching | `git-workflow-and-versioning` |
| CI/CD pipeline work | `ci-cd-and-automation` |
| Documentation or ADRs | `documentation-and-adrs` |
| Deploy or launch | `shipping-and-launch` |
| Removing old systems | `deprecation-and-migration` |
| Providing context to an agent | `context-engineering` |
| Grounding in official docs | `source-driven-development` |

### StackRox-specific tasks

| Intent | Skill(s) |
|--------|----------|
| Any StackRox UI change | `stackrox-ui-conventions` (always) |
| Building a PatternFly component | `patternfly-development` + `frontend-ui-engineering` |
| Fetching or mutating REST data for a new feature | `rest-and-service-layer` |
| Modifying an existing GraphQL query or mutation | `graphql-and-data-layer` |
| Wiring Redux or saga state | `react-state-patterns` |
| Cypress E2E test | `cypress-e2e-testing` |
| PatternFly major version upgrade | `deprecation-and-migration` + `patternfly-development` |
| Hardening a form or auth flow | `security-and-hardening` + `references/security-checklist.md` |
| Debugging in the browser | `debugging-and-error-recovery` + `browser-testing-with-devtools` |
| Reviewing a StackRox PR | `code-review-and-quality` + `agents/frontend-reviewer.md` |
| Onboarding to the codebase | `agents/stackrox-ui-guide.md` + `stackrox-ui-conventions` |

## Lifecycle Mapping

For agents without slash commands, follow this internal lifecycle:

- DEFINE → `idea-refine` (if fuzzy) → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development` + `stackrox-ui-conventions` (StackRox) + domain skill (e.g. `patternfly-development`, `rest-and-service-layer`, `graphql-and-data-layer`)
- VERIFY → `debugging-and-error-recovery`, `browser-testing-with-devtools`, `cypress-e2e-testing`
- REVIEW → `code-review-and-quality` + `security-and-hardening` + `performance-optimization`
- SHIP → `shipping-and-launch` + `git-workflow-and-versioning`

## Execution Model

For every request:

1. Determine which skill(s) apply (StackRox UI work always includes `stackrox-ui-conventions`).
2. Load the relevant `SKILL.md` files into context.
3. Follow each skill's workflow strictly, including Verification.
4. Only mark work complete after Verification passes.

## Anti-Rationalization

These thoughts are wrong and must be ignored:

- "This is too small for a skill"
- "I can just quickly implement this"
- "I will gather context later"
- "The verification step is optional if the change is simple"
- "I know this pattern -- I do not need to check the real StackRox codebase"

Correct behavior: check for skills first, apply them fully, verify against the real codebase when asserting StackRox facts.

## Slash Commands (Claude Code)

If your agent supports slash commands, these are wired up in `.claude/commands/`:

| Command | Skill(s) invoked |
|---------|------------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/code-simplify` | code-simplification |
| `/ship` | shipping-and-launch |

## Adding New Skills

See `CONTRIBUTING.md` for the full process. Short version:

- Create `skills/<kebab-case>/SKILL.md`.
- Add YAML frontmatter: `name` (matching the directory) and `description` (what it does, then "Use when...", max 1024 chars).
- Include all six required sections: Overview, When to Use, Core Process, Common Rationalizations, Red Flags, Verification.
- Ground StackRox-specific claims in verified facts from `ui/apps/platform/src/`.
- Cross-reference related skills rather than duplicating content.

## StackRox Directory Ground Truth

When asserting anything about StackRox UI structure, remember:

- Lives at `ui/apps/platform/src/`
- **PascalCase**: `Components/`, `Containers/`
- **lowercase**: `hooks/`, `services/`, `queries/`, `reducers/`, `sagas/`, `providers/`, `utils/`, `types/`, `constants/`, `sorters/`
- Route constants: `routePaths.ts` at the top level
- Verify any other claim against the real repo before committing.
