---
description: Start spec-driven development — write a structured specification before writing code
---

Invoke the agent-skills:spec-driven-development skill.

Begin by understanding what the user wants to build. Ask clarifying questions about:
1. The objective and target users
2. Core features and acceptance criteria
3. Tech stack preferences and constraints
4. Known boundaries (what to always do, ask first about, and never do)

Then generate a structured spec covering all six core areas: objective, commands, project structure, code style, testing strategy, and boundaries.

If the work touches the StackRox UI codebase, also load agent-skills:stackrox-ui-conventions so the spec's project structure, code style, and boundaries sections reflect StackRox directory layout, naming, and data-layer defaults (REST-first via `services/`, GraphQL only for existing or backend-authored endpoints).

Save the spec as SPEC.md in the project root and confirm with the user before proceeding.
