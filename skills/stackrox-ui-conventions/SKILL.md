---
name: stackrox-ui-conventions
description: Enforces StackRox UI project conventions for directory structure, naming, and component organization. Use when creating new features, adding components, or restructuring existing code in the StackRox UI codebase.
---

# StackRox UI Conventions

## Overview

StackRox UI follows a consistent set of conventions for organizing code, naming files, and composing components. Following these conventions keeps the codebase navigable across hundreds of features and ensures AI agents generate code that fits naturally into the existing structure.

## When to Use

- Creating a new feature or page in StackRox UI
- Adding a new component, hook, or service
- Deciding where a file belongs in the directory tree
- Reviewing code for structural consistency
- Onboarding to the StackRox UI codebase
- NOT for styling decisions (see `patternfly-development`)
- NOT for state management choices (see `react-state-patterns`)

## Directory Structure

The StackRox UI lives under `ui/apps/platform/src/`. Key directories:

```
ui/apps/platform/src/
  components/        # Shared, reusable UI components (buttons, modals, tables)
  containers/        # Page-level components connected to data (routes map here)
  services/          # API call wrappers (REST and GraphQL client functions)
  hooks/             # Shared custom React hooks
  reducers/          # Redux reducers (one file per slice)
  sagas/             # Redux-saga side effects
  queries/           # GraphQL query/mutation documents (.graphql or co-located)
  providers/         # React context providers (auth, theme, feature flags)
  types/             # Shared TypeScript type definitions
  utils/             # Pure utility functions (formatting, parsing, validation)
  constants/         # Shared constants and enums
  images/            # Static image assets
  routePaths.ts      # Centralized route path constants
```

### Placement Rules

```
Is it a full page with a route?        --> containers/
Is it a shared visual component?        --> components/
Is it a feature-scoped component?       --> containers/<Feature>/
Does it call an API?                    --> services/
Is it a GraphQL document?              --> queries/ or co-located with component
Is it a reusable React hook?           --> hooks/
Is it a Redux state slice?             --> reducers/
Is it a side effect (async workflow)?  --> sagas/
Is it a React context provider?        --> providers/
Is it a shared TypeScript type?        --> types/
Is it a pure utility function?         --> utils/
```

## Naming Conventions

### Files and Directories

| Type | Convention | Example |
|------|-----------|---------|
| Component file | PascalCase | `VulnerabilityTable.tsx` |
| Component directory | PascalCase | `VulnerabilityTable/` |
| Hook file | camelCase, `use` prefix | `useVulnerabilityData.ts` |
| Service file | camelCase | `vulnerabilityService.ts` |
| Reducer file | camelCase | `vulnerabilityReducer.ts` |
| Saga file | camelCase | `vulnerabilitySaga.ts` |
| Type file | camelCase | `vulnerabilityTypes.ts` |
| Utility file | camelCase | `vulnerabilityUtils.ts` |
| Test file | same as source + `.test` | `VulnerabilityTable.test.tsx` |
| GraphQL document | camelCase | `getVulnerabilities.graphql` |
| Constant file | camelCase | `vulnerabilityConstants.ts` |

### Exports and Symbols

| Type | Convention | Example |
|------|-----------|---------|
| React component | PascalCase, named export | `export function VulnerabilityTable()` |
| Hook | camelCase, `use` prefix | `export function useVulnerabilityData()` |
| Service function | camelCase, verb prefix | `export function fetchVulnerabilities()` |
| Type / Interface | PascalCase | `export type VulnerabilityRow` |
| Enum | PascalCase | `export enum VulnerabilitySeverity` |
| Constant | SCREAMING_SNAKE_CASE | `export const MAX_RETRY_COUNT = 3` |
| Redux action type | SCREAMING_SNAKE_CASE | `FETCH_VULNERABILITIES_SUCCESS` |
| Redux action creator | camelCase | `fetchVulnerabilitiesSuccess()` |

## Feature Organization

A feature in StackRox UI groups related components, hooks, and logic under the container directory:

```
containers/
  Vulnerabilities/
    VulnerabilitiesPage.tsx          # Route entry point
    VulnerabilityTable.tsx           # Feature-specific component
    VulnerabilityDetailPanel.tsx     # Feature-specific component
    useVulnerabilityFilters.ts       # Feature-specific hook
    vulnerabilityTypes.ts            # Feature-specific types
    constants.ts                     # Feature-specific constants
    __tests__/
      VulnerabilitiesPage.test.tsx
      VulnerabilityTable.test.tsx
```

### When to Extract to Shared Directories

```
Is the component used by 2+ features?
  YES --> Move to components/
  NO  --> Keep in containers/<Feature>/

Is the hook used by 2+ features?
  YES --> Move to hooks/
  NO  --> Keep in containers/<Feature>/

Is the type used by 2+ features?
  YES --> Move to types/
  NO  --> Keep in containers/<Feature>/
```

Rule: start feature-scoped, promote to shared only when reuse is real (not hypothetical).

## Component Composition Rules

### Container / Presentation Split

Containers connect to data (Redux, Apollo, services). Presentation components receive props and render UI.

```tsx
// Container: containers/Vulnerabilities/VulnerabilitiesPage.tsx
export function VulnerabilitiesPage() {
  const { data, loading, error } = useQuery(GET_VULNERABILITIES);
  const filters = useVulnerabilityFilters();

  if (loading) return <Bullseye><Spinner /></Bullseye>;
  if (error) return <ErrorState error={error} />;

  return (
    <VulnerabilityTable
      rows={data.vulnerabilities}
      filters={filters}
    />
  );
}

// Presentation: containers/Vulnerabilities/VulnerabilityTable.tsx
export function VulnerabilityTable({ rows, filters }: VulnerabilityTableProps) {
  // Pure rendering -- no data fetching, no side effects
  return (
    <Table aria-label="Vulnerabilities">
      {/* ... */}
    </Table>
  );
}
```

### Composition Over Configuration

Prefer composing PatternFly components over wrapping them with configuration props:

```tsx
// Preferred: composition
<PageSection variant="light">
  <TextContent>
    <Text component="h1">Vulnerabilities</Text>
  </TextContent>
</PageSection>

// Avoid: wrapper with config props
<PageHeader title="Vulnerabilities" variant="light" />
```

### Index Files

Use `index.ts` barrel files sparingly. Prefer direct imports to avoid circular dependencies:

```tsx
// Preferred: direct import
import { VulnerabilityTable } from './VulnerabilityTable';

// Avoid: barrel import (hides dependency graph, causes circular imports)
import { VulnerabilityTable } from './';
```

## Route Registration

Routes map to container components in the routing configuration:

```tsx
// Route paths are centralized in routePaths.ts
export const vulnerabilitiesPath = '/main/vulnerabilities';

// Route registration connects path to container
<Route path={vulnerabilitiesPath} component={VulnerabilitiesPage} />
```

New pages require:
1. A route path constant in `routePaths.ts`
2. A container component in `containers/`
3. A `<Route>` entry in the appropriate router configuration

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll put everything in components/ for now" | Feature-scoped code belongs in containers/. Premature sharing creates coupling between unrelated features. |
| "Index files make imports cleaner" | They hide the dependency graph, cause circular imports, and make tree-shaking harder. Use direct imports. |
| "This component is small enough to skip the container/presentation split" | The split isn't about size -- it's about separating data concerns from rendering. Even small components benefit. |
| "I'll create a shared hook for this since it might be reused" | Start feature-scoped. Extract to hooks/ only when a second feature actually needs it. |
| "The naming convention doesn't matter for internal files" | Inconsistent naming makes grep useless. Follow the convention even for files you think nobody else will touch. |

## Red Flags

- Components in `components/` that are only used by one feature (should be in `containers/<Feature>/`)
- Service functions inside component files (should be in `services/`)
- GraphQL queries defined inline in components (should be in `queries/` or a co-located file)
- Redux actions, reducers, and sagas in the same file (separate by concern)
- Barrel `index.ts` files re-exporting everything from a directory
- Route paths hardcoded as string literals instead of using `routePaths.ts`
- Feature-specific types in the shared `types/` directory

## Verification

After creating or restructuring code in StackRox UI:

- [ ] New files are in the correct directory per the placement rules
- [ ] File and symbol names follow the naming conventions table
- [ ] Feature-scoped code lives under `containers/<Feature>/`, not in shared directories
- [ ] Container components handle data; presentation components receive props
- [ ] Route paths use constants from `routePaths.ts`
- [ ] No barrel `index.ts` files added
- [ ] Shared code is actually used by 2+ features
- [ ] Cross-references to `patternfly-development` and `react-state-patterns` are consistent
