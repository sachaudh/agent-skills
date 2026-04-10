# StackRox Frontend Agent Skills

**A portable AI brain for StackRox frontend engineering.**

Production-grade skills, agent personas, and references tailored for React 18.2, TypeScript 5.9, PatternFly 6, Apollo Client 3.8, Redux 4 + saga, and the StackRox UI codebase (`ui/apps/platform/src/`). Works with any AI coding agent that accepts Markdown instructions.

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## Target Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 18.2 |
| Language | TypeScript 5.9 |
| Component library | PatternFly 6 (react-core, react-table, react-charts, react-topology, react-icons) |
| Data layer | Axios (REST primary, via `services/` + `useRestQuery` / `useRestMutation` / `usePaginatedQuery`), Apollo Client 3.8 (GraphQL for existing queries and new backend-authored GraphQL endpoints) |
| State management | Redux 4 + redux-thunk + redux-saga, React Router 5 |
| Forms | Formik 2 + Yup, redux-form (legacy) |
| Styling | Tailwind CSS, styled-components 5, CSS custom properties (light/dark themes) |
| Build | Vite 6, Webpack 5 (OpenShift Console plugin only) |
| Unit/component tests | Vitest 4 + @testing-library/react 16 |
| E2E tests | Cypress 15 |
| Linting | ESLint 9 + @typescript-eslint 8, Prettier |
| Package manager | npm |
| Node | >= 22.13.0 |

---

## Commands

7 slash commands that map to the development lifecycle. Each activates the right skills automatically.

| What you are doing | Command | Key principle |
|--------------------|---------|---------------|
| Define what to build | `/spec` | Spec before code |
| Plan how to build it | `/plan` | Small, atomic tasks |
| Build incrementally | `/build` | One slice at a time |
| Prove it works | `/test` | Tests are proof |
| Review before merge | `/review` | Improve code health |
| Simplify the code | `/code-simplify` | Clarity over cleverness |
| Ship to production | `/ship` | Faster is safer |

---

## Quick Start

Clone and reference the skills from your agent of choice. This is documentation -- there is no install step.

```bash
git clone https://github.com/sachaudh/agent-skills.git
```

**Claude Code:** Add the repo path to your plugin directory, or paste the relevant `SKILL.md` content into your project's `CLAUDE.md`.

**Cursor / Copilot / Gemini CLI / OpenCode / Windsurf:** Skills are plain Markdown -- copy any `SKILL.md` into your rules file, system prompt, or project context. See `docs/getting-started.md`.

**Any agent:** Reference the skill by name in conversation: "Follow the `stackrox-ui-conventions` process for this change."

---

## All 27 Skills

Skills are grouped by development phase. Load only the ones relevant to the current task -- loading everything wastes context.

### Meta

| Skill | What it does | Use when |
|-------|--------------|----------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | Discovery flowchart that maps task types to the right skill | Starting a session or switching tasks |

### Define

| Skill | What it does | Use when |
|-------|--------------|----------|
| [idea-refine](skills/idea-refine/SKILL.md) | Structured divergent/convergent thinking to turn vague ideas into concrete proposals | You have a rough concept that needs exploration |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Write a PRD covering objectives, commands, structure, code style, testing, and boundaries before any code | Starting a new project, feature, or significant change |

### Plan

| Skill | What it does | Use when |
|-------|--------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Decompose specs into small, verifiable tasks with acceptance criteria and dependency ordering | You have a spec and need implementable units |

### Build -- core

| Skill | What it does | Use when |
|-------|--------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Thin vertical slices -- implement, test, verify, commit | Any change touching more than one file |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Red-Green-Refactor, test pyramid, DAMP over DRY | Implementing logic, fixing bugs, or changing behavior |
| [context-engineering](skills/context-engineering/SKILL.md) | Feed agents the right information at the right time | Starting a session, switching tasks, or when output quality drops |
| [source-driven-development](skills/source-driven-development/SKILL.md) | Ground every framework decision in official documentation | You want authoritative, source-cited code |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | Component architecture, responsive design, WCAG 2.1 AA accessibility | Building or modifying user-facing interfaces |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | Contract-first design, Hyrum's Law, One-Version Rule, error semantics | Designing APIs, module boundaries, or public interfaces |

### Build -- StackRox specific

| Skill | What it does | Use when |
|-------|--------------|----------|
| [stackrox-ui-conventions](skills/stackrox-ui-conventions/SKILL.md) | Directory structure (`Components/`, `Containers/`, `hooks/`, ...), naming, feature organization, composition rules | Any StackRox UI change -- load this alongside the domain skill |
| [patternfly-development](skills/patternfly-development/SKILL.md) | PF6 component selection, theming, accessibility, wrap-vs-direct decisions | Building or modifying PatternFly components |
| [rest-and-service-layer](skills/rest-and-service-layer/SKILL.md) | REST-first data fetching via `services/` + `useRestQuery` / `useRestMutation` / `usePaginatedQuery`, cancellation, search, error formatting | Fetching or mutating server data for a new feature |
| [graphql-and-data-layer](skills/graphql-and-data-layer/SKILL.md) | Apollo Client 3.8 query/mutation patterns, cache policies, optimistic updates | Modifying existing GraphQL or building UI for a new GraphQL endpoint |
| [react-state-patterns](skills/react-state-patterns/SKILL.md) | Decision tree for Apollo cache, useState, Context, Redux + saga | Choosing or migrating state management for a feature |

### Verify

| Skill | What it does | Use when |
|-------|--------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Chrome DevTools MCP for live runtime data -- DOM, console, network, performance | Building or debugging anything that runs in a browser |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | Five-step triage: reproduce, localize, reduce, fix, guard | Tests fail, builds break, or behavior is unexpected |
| [cypress-e2e-testing](skills/cypress-e2e-testing/SKILL.md) | Page objects, `cy.intercept()` for GraphQL, `cy.session()` auth, PF6 testing with `ouiaId` | Writing or updating StackRox E2E tests |

### Review

| Skill | What it does | Use when |
|-------|--------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Five-axis review, change sizing, severity labels, review speed norms | Before merging any change |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton's Fence, Rule of 500, reduce complexity while preserving behavior | Code works but is harder to read or maintain than it should be |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | React XSS, `dangerouslySetInnerHTML`, Formik/Yup, CSP, auth token handling | Handling user input, auth, or rendering untrusted content |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Measure-first approach, Core Web Vitals, profiling, bundle analysis | Performance requirements exist or you suspect regressions |

### Ship

| Skill | What it does | Use when |
|-------|--------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | Trunk-based development, atomic commits, commit-as-save-point | Making any code change |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | Shift Left, Faster is Safer, feature flags, quality gate pipelines | Setting up or modifying build and deploy pipelines |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | PatternFly major upgrades, React Router v5→v6, redux-form→Formik, codemods | Removing old systems or migrating patterns |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Architecture Decision Records, API docs, inline documentation -- document the *why* | Making architectural decisions or changing APIs |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | Pre-launch checklists, feature flag lifecycle, staged rollouts, rollback | Preparing to deploy to production |

---

## Agent Personas

Pre-configured specialist personas for targeted reviews:

| Agent | Role | Perspective |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | Senior Staff Engineer | Five-axis generic code review |
| [frontend-reviewer](agents/frontend-reviewer.md) | Senior React/TS Engineer | Six-dimension PatternFly-aware review |
| [stackrox-ui-guide](agents/stackrox-ui-guide.md) | Onboarding guide | Explains StackRox UI directory structure and data flow |
| [security-auditor](agents/security-auditor.md) | Security Engineer | Vulnerability detection, threat modeling |
| [test-engineer](agents/test-engineer.md) | QA Specialist | Test strategy, coverage analysis, Prove-It pattern |

---

## Reference Checklists

Quick-reference material that skills pull in when needed:

| Reference | Covers |
|-----------|--------|
| [testing-patterns.md](references/testing-patterns.md) | Test structure, naming, mocking, React/API/E2E examples, anti-patterns |
| [security-checklist.md](references/security-checklist.md) | Frontend auth (httpOnly cookies, Apollo token refresh), Formik/Yup validation, React-specific items |
| [performance-checklist.md](references/performance-checklist.md) | Core Web Vitals targets, frontend checklists, measurement commands |
| [accessibility-checklist.md](references/accessibility-checklist.md) | Keyboard nav, screen readers, visual design, ARIA, testing tools |
| [patternfly-checklist.md](references/patternfly-checklist.md) | PF6 component selection, theming, accessibility, deprecated replacements, imports |
| [stackrox-ui-patterns.md](references/stackrox-ui-patterns.md) | Table page, detail panel, form wizard, filter bar, empty state, error boundary patterns |

---

## How Skills Work

Every skill follows a consistent anatomy:

```
┌─────────────────────────────────────────────┐
│  SKILL.md                                   │
│                                             │
│  ┌─ Frontmatter ─────────────────────────┐  │
│  │ name: lowercase-hyphen-name           │  │
│  │ description: Use when [trigger]       │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Overview              → What this skill does │
│  When to Use           → Triggering conditions │
│  Core Process          → Step-by-step workflow │
│  Common Rationalizations → Excuses + rebuttals │
│  Red Flags             → Signs something is wrong │
│  Verification          → Evidence requirements │
└─────────────────────────────────────────────┘
```

**Key design choices:**

- **Process, not prose.** Skills are workflows agents follow, not reference docs they read. Each has steps, checkpoints, and exit criteria.
- **Anti-rationalization.** Every skill includes a table of common excuses agents use to skip steps with documented counter-arguments.
- **Verification is non-negotiable.** Every skill ends with evidence requirements -- tests passing, build output, runtime data. "Seems right" is never sufficient.
- **Progressive disclosure.** The `SKILL.md` is the entry point. References load only when needed, keeping token usage minimal.

---

## Project Structure

```
agent-skills/
├── skills/                             # 27 core skills (SKILL.md per directory)
│   ├── using-agent-skills/             #   Meta: discovery flowchart
│   ├── idea-refine/                    #   Define
│   ├── spec-driven-development/        #   Define
│   ├── planning-and-task-breakdown/    #   Plan
│   ├── incremental-implementation/     #   Build
│   ├── test-driven-development/        #   Build
│   ├── context-engineering/            #   Build (StackRox-customized)
│   ├── source-driven-development/      #   Build
│   ├── frontend-ui-engineering/        #   Build
│   ├── api-and-interface-design/       #   Build
│   ├── stackrox-ui-conventions/        #   Build -- StackRox (new)
│   ├── patternfly-development/         #   Build -- StackRox (new)
│   ├── rest-and-service-layer/         #   Build -- StackRox (new)
│   ├── graphql-and-data-layer/         #   Build -- StackRox (new)
│   ├── react-state-patterns/           #   Build -- StackRox (new)
│   ├── browser-testing-with-devtools/  #   Verify
│   ├── debugging-and-error-recovery/   #   Verify
│   ├── cypress-e2e-testing/            #   Verify -- StackRox (new)
│   ├── code-review-and-quality/        #   Review
│   ├── code-simplification/            #   Review
│   ├── security-and-hardening/         #   Review (StackRox-customized)
│   ├── performance-optimization/       #   Review
│   ├── git-workflow-and-versioning/    #   Ship
│   ├── ci-cd-and-automation/           #   Ship
│   ├── deprecation-and-migration/      #   Ship (StackRox-customized)
│   ├── documentation-and-adrs/         #   Ship
│   └── shipping-and-launch/            #   Ship
├── agents/                             # 5 specialist personas
├── references/                         # 6 supplementary checklists
├── hooks/                              # SIMPLIFY-IGNORE.md (reference doc)
├── .claude/commands/                   # 7 slash commands
└── docs/                               # getting-started.md, skill-anatomy.md
```

---

## Why StackRox Frontend Agent Skills?

AI coding agents default to the shortest path -- which often means skipping specs, tests, security reviews, and the practices that make software reliable. A portable brain with structured workflows enforces the same discipline senior engineers bring to production code.

This fork narrows the focus to StackRox frontend engineering. Generic skills cover the engineering lifecycle; StackRox-specific skills encode the tech stack, directory conventions, and component patterns of `ui/apps/platform/src/`. Every StackRox claim is verified against the real repo -- and when the real repo drifts, the skills get updated to match.

---

## Contributing

Skills should be **specific** (actionable steps, not vague advice), **verifiable** (clear exit criteria with evidence requirements), **battle-tested** (based on real workflows), **minimal** (only what is needed to guide the agent), and **grounded** (StackRox-specific claims verified against the real codebase).

See [docs/skill-anatomy.md](docs/skill-anatomy.md) for the format specification and [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT -- use these skills in your projects, teams, and tools.
