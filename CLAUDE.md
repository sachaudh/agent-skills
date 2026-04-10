# StackRox Frontend Agent Skills

A portable AI brain for StackRox frontend engineering -- production-grade skills, agents, and references tailored for React 18.2, TypeScript 5.9, PatternFly 6, Apollo Client 3.8, Redux 4, and the StackRox UI codebase.

This repo is documentation-only (Markdown + YAML frontmatter). It is consumed by AI coding agents (Claude Code, Cursor, Copilot, Gemini CLI, etc.) as a portable plugin.

## Target Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 18.2 |
| Language | TypeScript 5.9 |
| Component library | PatternFly 6 (react-core, react-table, react-charts, react-topology, react-icons) |
| Data layer | Axios (REST primary, via `services/` + `useRestQuery` / `useRestMutation` / `usePaginatedQuery`), Apollo Client 3.8 (GraphQL for existing queries and new backend-authored GraphQL endpoints) |
| State management | Redux 4 + redux-thunk + redux-saga, React Router 5 (MobX 6 present only as a transitive peer dep of `@patternfly/react-topology`) |
| Forms | Formik 2 + Yup, redux-form (legacy) |
| Styling | Tailwind CSS, styled-components 5, PostCSS, CSS custom properties (light/dark themes) |
| Build | Vite 6, Webpack 5 (OpenShift Console plugin only) |
| Unit/component tests | Vitest 4 + @testing-library/react 16 |
| E2E tests | Cypress 15 |
| Linting | ESLint 9 + @typescript-eslint 8, Prettier |
| Package manager | npm |
| Node | >= 22.13.0 |

## Project Structure

```
skills/            → 27 core skills (SKILL.md per directory)
agents/            → 5 agent personas (code-reviewer, frontend-reviewer,
                     security-auditor, stackrox-ui-guide, test-engineer)
references/        → 6 supplementary checklists (accessibility, patternfly,
                     performance, security, stackrox-ui-patterns, testing)
hooks/             → SIMPLIFY-IGNORE.md (reference doc for the simplify-ignore pattern)
docs/              → getting-started.md, skill-anatomy.md
.claude/commands/  → Slash commands (/spec, /plan, /build, /test, /review,
                     /code-simplify, /ship)
tasks/             → Living artifacts (plan.md, todo.md) created by /plan
SPEC.md            → Living spec created by /spec
```

## Skills by Phase

**Meta:** using-agent-skills (discovery flowchart), idea-refine (fuzzy-idea → spec input)

**Define:** spec-driven-development

**Plan:** planning-and-task-breakdown

**Build:**
- Core: incremental-implementation, test-driven-development, context-engineering, source-driven-development, api-and-interface-design, frontend-ui-engineering
- StackRox: stackrox-ui-conventions, patternfly-development, rest-and-service-layer, graphql-and-data-layer, react-state-patterns

**Verify:** browser-testing-with-devtools, debugging-and-error-recovery, cypress-e2e-testing

**Review:** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization

**Ship:** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, shipping-and-launch

## Agents

| Agent | Purpose |
|-------|---------|
| `agents/code-reviewer.md` | Generic five-axis code review |
| `agents/frontend-reviewer.md` | React/TypeScript/PatternFly six-dimension review |
| `agents/security-auditor.md` | Vulnerability detection |
| `agents/stackrox-ui-guide.md` | Onboarding persona -- explains StackRox UI directory structure and data flow |
| `agents/test-engineer.md` | Test strategy and writing |

## References

| Reference | Used with |
|-----------|-----------|
| `references/accessibility-checklist.md` | frontend-ui-engineering, patternfly-development |
| `references/patternfly-checklist.md` | patternfly-development |
| `references/performance-checklist.md` | performance-optimization |
| `references/security-checklist.md` | security-and-hardening (frontend-focused) |
| `references/stackrox-ui-patterns.md` | stackrox-ui-conventions (table pages, detail panels, wizards, filter bars, empty states, error boundaries) |
| `references/testing-patterns.md` | test-driven-development |

## Conventions

- Every skill lives in `skills/<name>/SKILL.md` with matching directory name.
- YAML frontmatter has `name` and `description` fields. Description starts with what the skill does (third person), followed by "Use when..." trigger conditions. Maximum 1024 characters.
- Every skill has the standard sections from `docs/skill-anatomy.md`: Overview, When to Use, Core Process, Common Rationalizations, Red Flags, Verification.
- References live in `references/`, not inside skill directories.
- Supporting files only created when content exceeds 100 lines; otherwise inline into `SKILL.md`.
- Cross-reference related skills instead of duplicating content.
- StackRox-specific skills ground themselves in durable conventions (tech stack, directory structure, component patterns) -- not team processes that change quarterly.

## Commands

This is a documentation project. There is no test runner or build step.

**Validation** (manual or scripted):
- Every `SKILL.md` has valid YAML frontmatter with `name` and `description`
- `name` in frontmatter matches the directory name
- Every skill has all six standard sections
- Cross-references resolve to existing files

## Boundaries

### Always
- Follow `docs/skill-anatomy.md` format for new skills.
- Keep skills actionable (concrete processes, not vague advice).
- Cross-reference related skills instead of duplicating content.
- Ground StackRox-specific content in durable conventions.

### Ask first
- Before removing a skill entirely (in case it has unexpected utility).
- Before encoding any process that might be team-specific or change quarterly.

### Never
- Hardcode internal URLs, secrets, or credentials.
- Include team-specific processes that change frequently (sprint cadence, on-call rotation, etc.).
- Add skills that are just tips and tricks without a structured process.
- Duplicate content that belongs in the StackRox repo's own CLAUDE.md or CONTRIBUTING.md.
- Add backend-specific content (bcrypt, SQL injection, server-side rate limiting) outside clearly labeled "backend reference" sections.
