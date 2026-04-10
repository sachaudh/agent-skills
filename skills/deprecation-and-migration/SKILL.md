---
name: deprecation-and-migration
description: Manages deprecation and migration for frontend systems. Use when removing old components, upgrading PatternFly versions, migrating between state management approaches, or modernizing React patterns. Use when deciding whether to maintain legacy frontend code or invest in migration.
---

# Deprecation and Migration

## Overview

Code is a liability, not an asset. Every line of code has ongoing maintenance cost — bugs to fix, dependencies to update, security patches to apply, and new engineers to onboard. Deprecation is the discipline of removing code that no longer earns its keep, and migration is the process of moving users safely from the old to the new.

Most engineering organizations are good at building things. Few are good at removing them. This skill addresses that gap.

## When to Use

- Replacing an old system, API, or library with a new one
- Sunsetting a feature that's no longer needed
- Consolidating duplicate implementations
- Removing dead code that nobody owns but everybody depends on
- Planning the lifecycle of a new system (deprecation planning starts at design time)
- Deciding whether to maintain a legacy system or invest in migration

## Core Principles

### Code Is a Liability

Every line of code has ongoing cost: it needs tests, documentation, security patches, dependency updates, and mental overhead for anyone working nearby. The value of code is the functionality it provides, not the code itself. When the same functionality can be provided with less code, less complexity, or better abstractions — the old code should go.

### Hyrum's Law Makes Removal Hard

With enough users, every observable behavior becomes depended on — including bugs, timing quirks, and undocumented side effects. This is why deprecation requires active migration, not just announcement. Users can't "just switch" when they depend on behaviors the replacement doesn't replicate.

### Deprecation Planning Starts at Design Time

When building something new, ask: "How would we remove this in 3 years?" Systems designed with clean interfaces, feature flags, and minimal surface area are easier to deprecate than systems that leak implementation details everywhere.

## The Deprecation Decision

Before deprecating anything, answer these questions:

```
1. Does this system still provide unique value?
   → If yes, maintain it. If no, proceed.

2. How many users/consumers depend on it?
   → Quantify the migration scope.

3. Does a replacement exist?
   → If no, build the replacement first. Don't deprecate without an alternative.

4. What's the migration cost for each consumer?
   → If trivially automated, do it. If manual and high-effort, weigh against maintenance cost.

5. What's the ongoing maintenance cost of NOT deprecating?
   → Security risk, engineer time, opportunity cost of complexity.
```

## Compulsory vs Advisory Deprecation

| Type | When to Use | Mechanism |
|------|-------------|-----------|
| **Advisory** | Migration is optional, old system is stable | Warnings, documentation, nudges. Users migrate on their own timeline. |
| **Compulsory** | Old system has security issues, blocks progress, or maintenance cost is unsustainable | Hard deadline. Old system will be removed by date X. Provide migration tooling. |

**Default to advisory.** Use compulsory only when the maintenance cost or risk justifies forcing migration. Compulsory deprecation requires providing migration tooling, documentation, and support — you can't just announce a deadline.

## The Migration Process

### Step 1: Build the Replacement

Don't deprecate without a working alternative. The replacement must:

- Cover all critical use cases of the old system
- Have documentation and migration guides
- Be proven in production (not just "theoretically better")

### Step 2: Announce and Document

```markdown
## Deprecation Notice: OldService

**Status:** Deprecated as of 2025-03-01
**Replacement:** NewService (see migration guide below)
**Removal date:** Advisory — no hard deadline yet
**Reason:** OldService requires manual scaling and lacks observability.
            NewService handles both automatically.

### Migration Guide
1. Replace `import { client } from 'old-service'` with `import { client } from 'new-service'`
2. Update configuration (see examples below)
3. Run the migration verification script: `npx migrate-check`
```

### Step 3: Migrate Incrementally

Migrate consumers one at a time, not all at once. For each consumer:

```
1. Identify all touchpoints with the deprecated system
2. Update to use the replacement
3. Verify behavior matches (tests, integration checks)
4. Remove references to the old system
5. Confirm no regressions
```

**The Churn Rule:** If you own the infrastructure being deprecated, you are responsible for migrating your users — or providing backward-compatible updates that require no migration. Don't announce deprecation and leave users to figure it out.

### Step 4: Remove the Old System

Only after all consumers have migrated:

```
1. Verify zero active usage (metrics, logs, dependency analysis)
2. Remove the code
3. Remove associated tests, documentation, and configuration
4. Remove the deprecation notices
5. Celebrate — removing code is an achievement
```

## Migration Patterns

### Strangler Pattern

Run old and new systems in parallel. Route traffic incrementally from old to new. When the old system handles 0% of traffic, remove it.

```
Phase 1: New system handles 0%, old handles 100%
Phase 2: New system handles 10% (canary)
Phase 3: New system handles 50%
Phase 4: New system handles 100%, old system idle
Phase 5: Remove old system
```

### Adapter Pattern

Create an adapter that translates calls from the old interface to the new implementation. Consumers keep using the old interface while you migrate the backend.

```typescript
// Adapter: old interface, new implementation
class LegacyTaskService implements OldTaskAPI {
  constructor(private newService: NewTaskService) {}

  // Old method signature, delegates to new implementation
  getTask(id: number): OldTask {
    const task = this.newService.findById(String(id));
    return this.toOldFormat(task);
  }
}
```

### Feature Flag Migration

Use feature flags to switch consumers from old to new system one at a time:

```typescript
function getTaskService(userId: string): TaskService {
  if (featureFlags.isEnabled('new-task-service', { userId })) {
    return new NewTaskService();
  }
  return new LegacyTaskService();
}
```

## Zombie Code

Zombie code is code that nobody owns but everybody depends on. It's not actively maintained, has no clear owner, and accumulates security vulnerabilities and compatibility issues. Signs:

- No commits in 6+ months but active consumers exist
- No assigned maintainer or team
- Failing tests that nobody fixes
- Dependencies with known vulnerabilities that nobody updates
- Documentation that references systems that no longer exist

**Response:** Either assign an owner and maintain it properly, or deprecate it with a concrete migration plan. Zombie code cannot stay in limbo — it either gets investment or removal.

## Frontend Migration Patterns

The StackRox UI has specific migration patterns that recur as the frontend stack evolves. For PatternFly component details, see the `patternfly-development` skill. For state management decisions, see the `react-state-patterns` skill.

### 1. PatternFly Major Version Upgrades

PatternFly upgrades often require component API changes. Use wrapper components to isolate the migration surface.

**Example: Select component migration (PF5 to PF6)**

```typescript
// BEFORE (PatternFly 5): Single Select with onChange string value
import { Select, SelectOption } from '@patternfly/react-core/deprecated';

<Select
  onSelect={(_event, value) => setSelected(value as string)}
  selections={selected}
  isOpen={isOpen}
  onToggle={setIsOpen}
>
  <SelectOption value="critical">Critical</SelectOption>
  <SelectOption value="high">High</SelectOption>
</Select>

// AFTER (PatternFly 6): Composable Select with MenuToggle
import { Select, SelectOption, SelectList, MenuToggle } from '@patternfly/react-core';

<Select
  isOpen={isOpen}
  onSelect={(_event, value) => setSelected(value as string)}
  onOpenChange={setIsOpen}
  toggle={(toggleRef) => (
    <MenuToggle ref={toggleRef} onClick={() => setIsOpen(!isOpen)} isExpanded={isOpen}>
      {selected || 'Select severity'}
    </MenuToggle>
  )}
>
  <SelectList>
    <SelectOption value="critical">Critical</SelectOption>
    <SelectOption value="high">High</SelectOption>
  </SelectList>
</Select>
```

**Migration strategy:**
1. Create a wrapper component (`SimpleSelect`) that encapsulates the new PF6 API
2. Replace old `Select` imports with the wrapper, one file at a time
3. Run Cypress tests after each batch of replacements
4. Remove the wrapper once all consumers are migrated (or keep it if it adds value)

### 2. React Router v5 to v6

StackRox currently uses React Router v5. Migration to v6 changes route definition and hook patterns.

```typescript
// BEFORE (React Router v5): render prop and useRouteMatch
import { Switch, Route, useRouteMatch, useHistory } from 'react-router-dom';

function VulnerabilityRoutes() {
  const { path } = useRouteMatch();
  const history = useHistory();

  return (
    <Switch>
      <Route exact path={path} component={VulnerabilityList} />
      <Route path={`${path}/:cveId`} component={VulnerabilityDetail} />
    </Switch>
  );
}

// AFTER (React Router v6): element prop, relative routes, useNavigate
import { Routes, Route, useNavigate } from 'react-router-dom';

function VulnerabilityRoutes() {
  const navigate = useNavigate();

  return (
    <Routes>
      <Route index element={<VulnerabilityList />} />
      <Route path=":cveId" element={<VulnerabilityDetail />} />
    </Routes>
  );
}
```

**Key changes:**
- `Switch` becomes `Routes`
- `component` prop becomes `element` with JSX
- `useHistory()` becomes `useNavigate()`
- `useRouteMatch()` becomes `useMatch()` or relative paths
- Nested routes are relative by default

### 3. redux-form to Formik

Migrate forms from redux-form (deprecated) to Formik + Yup step-by-step:

```
Step 1: Add Formik + Yup to the feature
  - Install if not already present
  - Create a Yup validation schema mirroring the redux-form validate function

Step 2: Rewrite the form component
  - Replace reduxForm() HOC with <Formik> component
  - Replace <Field component={...}> with Formik <Field> or useField()
  - Replace handleSubmit prop with Formik's onSubmit
  - Map existing validation logic to Yup schema

Step 3: Remove Redux form state
  - Remove the form reducer from the Redux store
  - Remove formValueSelector calls -- use Formik's values instead
  - Remove any dispatch(change(...)) calls -- use Formik's setFieldValue

Step 4: Verify
  - All form validation works (required fields, format checks)
  - Submit sends correct data
  - Error states display correctly
  - Form reset/clear works
```

### 4. Codemod and ESLint Automation

For large-scale migrations, use `jscodeshift` codemods and ESLint rules to automate repetitive changes.

```bash
# Run a PatternFly codemod across the codebase
npx jscodeshift -t ./codemods/pf5-to-pf6-select.ts \
  --extensions=tsx,ts \
  ui/apps/platform/src/**/*.tsx

# Add an ESLint rule to flag deprecated imports
# .eslintrc.js
{
  "rules": {
    "no-restricted-imports": ["error", {
      "paths": [{
        "name": "@patternfly/react-core/deprecated",
        "message": "Use @patternfly/react-core instead. See patternfly-development skill."
      }]
    }]
  }
}
```

**When to use codemods vs manual migration:**
- **Codemod:** Mechanical 1:1 replacements (import paths, renamed props, simple API changes)
- **Manual:** Behavioral changes, new component composition patterns, logic changes
- **ESLint rule:** Prevent regression after migration (flag deprecated imports in new code)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It still works, why remove it?" | Working code that nobody maintains accumulates security debt and complexity. Maintenance cost grows silently. |
| "Someone might need it later" | If it's needed later, it can be rebuilt. Keeping unused code "just in case" costs more than rebuilding. |
| "The migration is too expensive" | Compare migration cost to ongoing maintenance cost over 2-3 years. Migration is usually cheaper long-term. |
| "We'll deprecate it after we finish the new system" | Deprecation planning starts at design time. By the time the new system is done, you'll have new priorities. Plan now. |
| "Users will migrate on their own" | They won't. Provide tooling, documentation, and incentives — or do the migration yourself (the Churn Rule). |
| "We can maintain both systems indefinitely" | Two systems doing the same thing is double the maintenance, testing, documentation, and onboarding cost. |

## Red Flags

- Deprecated systems with no replacement available
- Deprecation announcements with no migration tooling or documentation
- "Soft" deprecation that's been advisory for years with no progress
- Zombie code with no owner and active consumers
- New features added to a deprecated system (invest in the replacement instead)
- Deprecation without measuring current usage
- Removing code without verifying zero active consumers

## Verification

After completing a deprecation:

- [ ] Replacement is production-proven and covers all critical use cases
- [ ] Migration guide exists with concrete steps and examples
- [ ] All active consumers have been migrated (verified by metrics/logs)
- [ ] Old code, tests, documentation, and configuration are fully removed
- [ ] No references to the deprecated system remain in the codebase
- [ ] Deprecation notices are removed (they served their purpose)
