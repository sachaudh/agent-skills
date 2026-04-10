# Contributing to StackRox Frontend Agent Skills

This repo is a portable AI brain for StackRox frontend engineering. It contains skills, agent personas, references, and slash commands tailored for React 18.2, TypeScript 5.9, PatternFly 6, Apollo Client 3.8, Redux 4, and the StackRox UI codebase. It is not a generic upstream pack -- contributions should assume that context.

## Before You Contribute

Read these first:

- `CLAUDE.md` -- project identity, conventions, boundaries
- `docs/skill-anatomy.md` -- the format specification for every skill
- `docs/getting-started.md` -- how skills are consumed

## Adding a New Skill

1. Create a directory under `skills/` with a kebab-case name.
2. Add a `SKILL.md` following `docs/skill-anatomy.md`.
3. Include YAML frontmatter with `name` and `description` fields. `name` must match the directory name.
4. Write the `description` as: what the skill does (third person), followed by "Use when..." trigger conditions. Maximum 1024 characters.

### Required Sections

Every `SKILL.md` must have these six sections, in this order, with these exact headings:

- **Overview** -- What this skill does and why it matters
- **When to Use** -- Triggering conditions
- **Core Process** -- Step-by-step workflow (use "Core Process", not "Process")
- **Common Rationalizations** -- Excuses agents use to skip steps, with rebuttals
- **Red Flags** -- Warning signs that the skill is being applied incorrectly
- **Verification** -- How to confirm the skill was applied correctly

### Skill Quality Bar

Skills must be:

- **Specific** -- actionable steps, not vague advice
- **Verifiable** -- clear exit criteria with evidence requirements
- **Battle-tested** -- based on real engineering workflows, not theoretical ideals
- **Minimal** -- only the content needed to guide the agent correctly
- **Grounded** -- StackRox-specific content ties to durable facts (tech stack, directory structure, component patterns), not team processes that change quarterly

### What Not to Do

- Do not duplicate content between skills -- cross-reference with inline skill names (e.g., "see `patternfly-development`").
- Do not add skills that are vague advice instead of structured processes.
- Do not create supporting files unless content exceeds 100 lines.
- Do not put reference material inside skill directories -- use `references/` instead.
- Do not hardcode internal URLs, secrets, or credentials.
- Do not add team-specific content that changes frequently (sprint cadence, on-call rotation).
- Do not add backend-only content (bcrypt, SQL injection, server-side rate limiting) outside clearly labeled "Backend Reference" sections. This repo is frontend-focused.
- Do not duplicate content that belongs in the StackRox repo's own `CLAUDE.md` or `CONTRIBUTING.md`.

## Adding a New Agent Persona

1. Create `agents/<name>.md` following the format of existing agents.
2. Include YAML frontmatter with `name` and `description` fields.
3. Cite the skills the agent should treat as authoritative (see `agents/frontend-reviewer.md` for an example).

## Adding a New Reference

1. Create `references/<name>-checklist.md` or `references/<name>-patterns.md`.
2. Reference it from the skills that should load it.
3. Update `CLAUDE.md` and `docs/getting-started.md` to list the new reference.

## Modifying Existing Skills

- Keep changes focused and minimal.
- Preserve the existing structure, section headings, and tone.
- Test that YAML frontmatter remains valid after edits.
- If the edit affects cross-references to other skills, grep for the skill name and update callers.
- If the edit changes a StackRox path or convention, verify the claim against `ui/apps/platform/src/` in the actual StackRox repo before committing.

## StackRox-Specific Grounding

When asserting anything about StackRox UI (directory names, file layouts, patterns), verify against the real codebase. Historical facts (legacy patterns, migration state) should be clearly labeled as such so they can be revisited as the repo evolves.

The StackRox UI lives at `ui/apps/platform/src/` and uses PascalCase for `Components/` and `Containers/`. Every other top-level directory is lowercase.

## Validation

This is a documentation project. There is no test runner. Before committing:

- Every `SKILL.md` must have valid YAML frontmatter with `name` and `description`.
- `name` in frontmatter must match the skill's directory name.
- Every skill must have all six required sections in order.
- Cross-references to other skills, agents, and references must resolve to existing files.

## Reporting Issues

Open an issue if you find:

- A skill that gives incorrect or outdated guidance
- A StackRox claim that does not match the real codebase
- Missing coverage for a common frontend workflow
- Inconsistencies between skills

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
