---
description: Break work into small verifiable tasks with acceptance criteria and dependency ordering
---

Invoke the agent-skills:planning-and-task-breakdown skill.

Read the existing spec (SPEC.md or equivalent) and the relevant codebase sections. Then:

1. Enter plan mode — read only, no code changes
2. Identify the dependency graph between components
3. Slice work vertically (one complete path per task, not horizontal layers)
4. Write tasks with acceptance criteria and verification steps
5. Add checkpoints between phases
6. Present the plan for human review

If the work touches the StackRox UI codebase, also load agent-skills:stackrox-ui-conventions before slicing tasks so vertical slices align with StackRox directory boundaries (`Containers/`, `services/`, `hooks/`, `types/`) and respect the REST-first data layer default. Reference agent-skills:rest-and-service-layer, agent-skills:graphql-and-data-layer, and agent-skills:react-state-patterns when a task involves data fetching or state decisions.

Save the plan to tasks/plan.md and task list to tasks/todo.md.
