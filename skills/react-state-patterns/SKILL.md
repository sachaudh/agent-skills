---
name: react-state-patterns
description: Guides state management decisions in StackRox UI's Apollo + Redux + React codebase. Use when choosing between local state, context, Redux, or Apollo cache, when adding new state to a feature, or when refactoring existing state management patterns.
---

# React State Patterns

## Overview

StackRox UI uses multiple state management approaches: Apollo Client cache for server data, React local state and context for UI state, and Redux 4 with thunk/saga for complex cross-cutting workflows. This skill provides a decision framework for choosing the right approach and patterns for working within each system without introducing inconsistency.

MobX appears in `package.json` and is imported in three files, but **MobX is not used as an application state management choice**. It is a transitive peer dependency of `@patternfly/react-topology`: `index.tsx` calls `mobxConfigure({ isolateGlobalState: true })` to prevent global-state conflicts, and two NetworkGraph files wrap topology components with `observer` because the PatternFly topology API requires it. Do not treat MobX as a state option when adding new state.

## When to Use

- Adding state to a new or existing feature
- Choosing between Apollo cache, useState, context, or Redux
- Working with Redux reducers, actions, or sagas
- Refactoring state management in an existing feature
- NOT for REST data fetching (see `rest-and-service-layer`)
- NOT for Apollo query/mutation patterns (see `graphql-and-data-layer`)
- NOT for project structure decisions (see `stackrox-ui-conventions`)

## State Decision Tree

Start here when adding state to a feature:

```
Is it server data fetched from an API?
  YES --> Use `rest-and-service-layer` for REST (default) or
          `graphql-and-data-layer` for existing GraphQL code
  NO  --> Is it local to a single component?
    YES --> useState or useReducer
    NO  --> Is it shared by 2-3 nearby components?
      YES --> Lift state to nearest common parent
      NO  --> Is it read-heavy, write-rare (theme, auth, feature flags)?
        YES --> React Context (via a provider in providers/)
        NO  --> Is it part of a complex async workflow (multi-step, orchestrated)?
          YES --> Redux + redux-saga (reducers/ + sagas/)
          NO  --> Redux + redux-thunk (reducers/)
```

### Quick Reference Table

| State Type | Tool | Location | Example |
|-----------|------|----------|---------|
| Server data | Apollo cache | queries/ | Vulnerability list, deployment details |
| Component UI state | useState | In component | Dropdown open, form input value |
| Complex component state | useReducer | In component | Multi-step wizard state |
| Shared UI state (2-3 components) | Lifted state | Parent component | Selected row in table + detail panel |
| App-wide read-heavy state | React Context | providers/ | Auth user, theme preference, feature flags |
| URL-shareable state | URL search params | In component | Filters, pagination, sort order |
| Complex async workflows | Redux + saga | reducers/ + sagas/ | Policy creation wizard, bulk operations |
| Simple async actions | Redux + thunk | reducers/ | Fetch-and-store patterns |

## React Local State Patterns

### useState for Simple State

```tsx
export function VulnerabilityFilterBar() {
  const [searchTerm, setSearchTerm] = useState('');
  const [isFilterOpen, setIsFilterOpen] = useState(false);

  return (
    <Toolbar>
      <ToolbarItem>
        <SearchInput
          value={searchTerm}
          onChange={(_event, value) => setSearchTerm(value)}
        />
      </ToolbarItem>
    </Toolbar>
  );
}
```

### useReducer for Complex Local State

```tsx
interface WizardState {
  step: number;
  formData: Partial<PolicyFormData>;
  validationErrors: Record<string, string>;
}

type WizardAction =
  | { type: 'NEXT_STEP' }
  | { type: 'PREV_STEP' }
  | { type: 'SET_FIELD'; field: string; value: unknown }
  | { type: 'SET_ERRORS'; errors: Record<string, string> };

function wizardReducer(state: WizardState, action: WizardAction): WizardState {
  switch (action.type) {
    case 'NEXT_STEP':
      return { ...state, step: state.step + 1 };
    case 'PREV_STEP':
      return { ...state, step: state.step - 1 };
    case 'SET_FIELD':
      return { ...state, formData: { ...state.formData, [action.field]: action.value } };
    case 'SET_ERRORS':
      return { ...state, validationErrors: action.errors };
    default:
      return state;
  }
}
```

### URL State for Shareable Filters

```tsx
import { useSearchParams } from 'react-router-dom';

export function useVulnerabilityFilters() {
  const [searchParams, setSearchParams] = useSearchParams();

  const filters = {
    query: searchParams.get('q') ?? '',
    severity: searchParams.get('severity') ?? '',
    page: Number(searchParams.get('page') ?? '1'),
  };

  function setFilter(key: string, value: string) {
    setSearchParams((prev) => {
      if (value) {
        prev.set(key, value);
      } else {
        prev.delete(key);
      }
      prev.set('page', '1'); // Reset pagination on filter change
      return prev;
    });
  }

  return { filters, setFilter };
}
```

## Redux Patterns

### Reducer Structure

One reducer per state slice, in `reducers/`:

```tsx
// reducers/vulnerabilityReducer.ts
import { Vulnerability } from '../types/vulnerability';

interface VulnerabilityState {
  items: Vulnerability[];
  isLoading: boolean;
  error: string | null;
  selectedIds: string[];
}

const initialState: VulnerabilityState = {
  items: [],
  isLoading: false,
  error: null,
  selectedIds: [],
};

export function vulnerabilityReducer(
  state = initialState,
  action: VulnerabilityAction
): VulnerabilityState {
  switch (action.type) {
    case 'FETCH_VULNERABILITIES_REQUEST':
      return { ...state, isLoading: true, error: null };
    case 'FETCH_VULNERABILITIES_SUCCESS':
      return { ...state, isLoading: false, items: action.payload };
    case 'FETCH_VULNERABILITIES_FAILURE':
      return { ...state, isLoading: false, error: action.payload };
    case 'SELECT_VULNERABILITY':
      return { ...state, selectedIds: [...state.selectedIds, action.payload] };
    case 'DESELECT_VULNERABILITY':
      return { ...state, selectedIds: state.selectedIds.filter((id) => id !== action.payload) };
    default:
      return state;
  }
}
```

### Redux-Saga for Complex Workflows

Use sagas for multi-step async operations with side effects:

```tsx
// sagas/vulnerabilitySaga.ts
import { call, put, takeLatest, select } from 'redux-saga/effects';
import { fetchVulnerabilities } from '../services/vulnerabilityService';

function* fetchVulnerabilitiesSaga(action: FetchVulnerabilitiesAction) {
  try {
    const filters = yield select((state) => state.vulnerabilities.activeFilters);
    const data = yield call(fetchVulnerabilities, { ...filters, ...action.payload });
    yield put({ type: 'FETCH_VULNERABILITIES_SUCCESS', payload: data });
  } catch (error) {
    yield put({
      type: 'FETCH_VULNERABILITIES_FAILURE',
      payload: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}

export function* vulnerabilitySaga() {
  yield takeLatest('FETCH_VULNERABILITIES_REQUEST', fetchVulnerabilitiesSaga);
}
```

### Redux-Thunk for Simple Async

Use thunks for straightforward fetch-and-store:

```tsx
// actions/vulnerabilityActions.ts
export function fetchVulnerabilities(filters: VulnerabilityFilters) {
  return async (dispatch: Dispatch) => {
    dispatch({ type: 'FETCH_VULNERABILITIES_REQUEST' });
    try {
      const data = await vulnerabilityService.fetchVulnerabilities(filters);
      dispatch({ type: 'FETCH_VULNERABILITIES_SUCCESS', payload: data });
    } catch (error) {
      dispatch({
        type: 'FETCH_VULNERABILITIES_FAILURE',
        payload: error instanceof Error ? error.message : 'Unknown error',
      });
    }
  };
}
```

## NetworkGraph and react-topology

`@patternfly/react-topology` (used for the network graph and other topology views) pulls in MobX as a peer dependency and exposes an API that expects its consumers to wrap certain components with `mobx-react`'s `observer`. This is the only reason MobX appears in the StackRox codebase.

When working inside `Containers/NetworkGraph/`:

- Use `observer` from `mobx-react` on topology subcomponents only when required by the react-topology API (e.g., custom edge, group, or node components). Follow the existing patterns in `StyleEdge.tsx` and `DefaultFakeGroup.tsx`.
- Do not create MobX stores for application state. Feature-level state (filters, selection, panels, fetched data) still belongs in Apollo cache, local state, or Redux per the decision tree above.
- Do not import from `mobx` directly in application code. The `mobxConfigure({ isolateGlobalState: true })` call in `src/index.tsx` exists solely to isolate react-topology's internal MobX instance; do not add to it.

## Anti-Mixing Rules

These rules prevent state management approaches from conflicting:

1. **Do not mix Redux and Apollo for the same data.** If data comes from a GraphQL endpoint, use Apollo cache. Do not fetch with Apollo and then store in Redux.

2. **Do not introduce MobX as an application state tool.** MobX exists only to satisfy `@patternfly/react-topology`. Never create MobX stores, observables, or actions for feature state, even inside NetworkGraph containers.

3. **Do not use React Context for frequently changing data.** Context re-renders all consumers on every change. Use it for stable values (auth, theme, feature flags), not for lists or counters.

4. **Do not create a Redux slice for state that only one component uses.** Use useState or useReducer instead.

5. **Do not store derived data.** Compute it:
   ```tsx
   // Wrong: storing derived state
   const [filteredItems, setFilteredItems] = useState([]);
   useEffect(() => {
     setFilteredItems(items.filter(matchesFilter));
   }, [items, filter]);

   // Correct: computing derived state
   const filteredItems = useMemo(
     () => items.filter(matchesFilter),
     [items, filter]
   );
   ```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll use Redux for everything -- it's consistent" | Redux adds boilerplate. Local state and Apollo cache handle 80% of cases with less code. |
| "MobX is in package.json so it's fair game" | MobX is a transitive peer dep of `@patternfly/react-topology`. StackRox owns zero MobX state. Use Apollo, local state, or Redux instead. |
| "I'll store the API response in Redux for caching" | Apollo Client already caches GraphQL responses. Duplicating data in Redux creates sync bugs. |
| "Context is fine for this list of items" | Context re-renders all consumers on every change. For dynamic data, use lifted state, Redux, or Apollo. |
| "useEffect to sync state is the React way" | useEffect for state sync is a code smell. Compute derived state with useMemo or restructure the component tree. |

## Red Flags

- Apollo query results being copied into Redux state
- `makeAutoObservable`, `makeObservable`, or new MobX stores anywhere in the codebase
- `observer` wrappers outside of react-topology subcomponents in `Containers/NetworkGraph/`
- Direct imports from `mobx` in application code
- React Context holding rapidly changing data (lists, counters, form state)
- useEffect used to synchronize two pieces of state
- Redux slices for state used by a single component
- Missing loading/error states in Redux async actions
- Derived state stored instead of computed

## Verification

After implementing state management:

- [ ] State tool matches the decision tree (Apollo for server data, local state for UI, etc.)
- [ ] No mixing of Apollo and Redux for the same data source
- [ ] No new MobX stores, observables, or imports from `mobx` in application code
- [ ] `observer` only used where react-topology requires it inside `Containers/NetworkGraph/`
- [ ] Context only used for stable, read-heavy values
- [ ] Derived state computed (useMemo), not stored (useState + useEffect)
- [ ] Redux actions handle loading, success, and error states
- [ ] Saga used for multi-step workflows, thunk for simple async
- [ ] URL search params used for user-shareable filter state
