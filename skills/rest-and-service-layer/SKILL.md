---
name: rest-and-service-layer
description: Implements StackRox UI's REST-first, service-layer-backed data fetching using the shared `axios` instance and the `useRestQuery`, `useRestMutation`, and `usePaginatedQuery` hooks. Use when fetching or mutating server data for a new feature, adding or extending a function in `services/`, wiring a component to loading/error/empty/data states, handling pagination or search filters via `searchOptionsToQuery`, formatting errors with `getAxiosErrorMessage`, adding cancellation for heavy or rapidly-changing requests, or deciding whether a feature should reach for REST or GraphQL. NOT for Apollo cache management or modifying existing GraphQL queries and mutations (see `graphql-and-data-layer`).
---

# REST and Service Layer

## Overview

StackRox UI is REST-first. Server data flows through a service function in `services/<Entity>Service.ts` that imports the shared `axios` instance from `./instance`, and components consume it via one of three hooks: `useRestQuery`, `useRestMutation`, or `usePaginatedQuery`. This skill covers the service-layer structure, the three hooks, cancellation, pagination, search, error formatting, and the REST-vs-GraphQL decision.

## When to Use

- Fetching server data for a new component, page, or feature
- Adding a new endpoint call (GET, POST, PUT, PATCH, DELETE) from the UI
- Wiring a component to loading, error, empty, and data states
- Building an infinite-scroll or "View more" list
- Cancelling in-flight requests on filter change or unmount
- Formatting a server error for display to the user
- Translating a filter-bar state into a server query string
- Deciding whether a feature should reach for REST or GraphQL
- NOT for writing or modifying Apollo queries, mutations, or cache policies (see `graphql-and-data-layer`)
- NOT for choosing between local component state, Redux, and server cache (see `react-state-patterns`)
- NOT for E2E interception of REST calls (see `cypress-e2e-testing`)

## Core Process

Follow these steps in order when adding or modifying a REST call.

1. **Search `services/` for an existing function.** Grep for the endpoint path and the entity name. Extend an existing service function before writing a new one. Duplication in the service layer shows up as the same URL built two different ways.
2. **Decide REST vs GraphQL** using the decision framework below. REST is the default. Stop and pick GraphQL only on the two named triggers.
3. **Add or extend the service function** in `services/<Entity>Service.ts`. Import `axios` from `./instance`, define a URL constant at the top of the file, write one exported function per operation, and return `response.data` (or the relevant nested field). Import request and response types from `types/<entity>Service.proto.ts` when a proto type exists.
4. **Consume it from the component** via `useRestQuery` (GET), `useRestMutation` (POST, PUT, PATCH, DELETE), or `usePaginatedQuery` (infinite-load lists). Never call `axios` directly from a component.
5. **Render all four states:** loading, error, empty, and data. Every `useRestQuery` call must handle the first three, not just the happy path.
6. **Format errors with `getAxiosErrorMessage`** from `utils/responseErrorUtils`. Do not show `error.message` raw to the user. The helper handles 403 messaging and nested `response.data.message` fields the StackRox API returns.
7. **Add cancellation** via `makeCancellableAxiosRequest` for heavy queries, rapidly-changing filters, or when a new request supersedes an in-flight one. Distinguish `CancelledPromiseError` from real errors so cancellation does not render an alert.

## REST vs GraphQL Decision

```
Is the feature fetching or mutating server data?
  │
  ├── Is there existing GraphQL code you are modifying?
  │     YES --> Use `graphql-and-data-layer` (Apollo)
  │     NO  --> continue
  │
  ├── Is the backend team adding a new GraphQL endpoint
  │   specifically for this feature?
  │     YES --> Use `graphql-and-data-layer` (Apollo)
  │     NO  --> continue
  │
  └── Use REST: add or extend a function in `services/`
      and consume it via the right hook.
```

REST is the default. GraphQL is only used when (a) you are editing existing GraphQL queries or mutations, or (b) the backend team is shipping a new GraphQL endpoint specifically for this feature. There is no third path. Do not add `tanstack-query`, `swr`, `react-query`, or any other data-fetching library. The three StackRox hooks are deliberately hand-rolled and `usePaginatedQuery` explicitly notes that the library choice is reserved for a future migration decision.

## Patterns

### Service layer structure

Service files live in `services/<Entity>Service.ts`. They import `axios` from the local `./instance` module, declare URL constants at the top, and export one function per operation. Real example from `services/ClustersService.ts`:

```ts
import qs from 'qs';

import searchOptionsToQuery from 'services/searchOptionsToQuery';
import type { RestSearchOption } from 'services/searchOptionsToQuery';
import type { ClustersResponse } from 'types/clusterService.proto';
import axios from './instance';
import type { Empty } from './types';

const clustersUrl = '/v1/clusters';

export function fetchClusters(options?: RestSearchOption[]): Promise<Cluster[]> {
    let queryString = '';
    if (options && options.length !== 0) {
        const query = searchOptionsToQuery(options);
        queryString = qs.stringify(
            { query },
            { addQueryPrefix: true, arrayFormat: 'repeat', allowDots: true }
        );
    }
    return axios
        .get<{ clusters: Cluster[] }>(`${clustersUrl}${queryString}`)
        .then((response) => response?.data?.clusters ?? []);
}
```

The shared `axios` instance is defined exactly once in `services/instance.js`, where the comment is explicit: "This is the one place where we're allowed to import directly from 'axios'. All other places must use the instance exported here." The instance adds a 10-second timeout and is the hook point for future interceptors.

### `useRestQuery` (GET)

Returns `{ data, isLoading, error, refetch }`. Handles mount and unmount safely, supports cancellable requests, and clears prior errors before each new request by default. Canonical example from `Containers/Clusters/InitBundles/InitBundlesPage.tsx`:

```tsx
import useRestQuery from 'hooks/useRestQuery';
import { fetchClusterInitBundles } from 'services/ClustersService';
import { getAxiosErrorMessage } from 'utils/responseErrorUtils';

const {
    data: dataForFetch,
    error: errorForFetch,
    isLoading: isFetching,
    refetch,
} = useRestQuery(fetchClusterInitBundles);

return isFetching ? (
    <Bullseye><Spinner /></Bullseye>
) : errorForFetch ? (
    <Alert variant="warning" title="Unable to fetch cluster init bundles" component="p" isInline>
        {getAxiosErrorMessage(errorForFetch)}
    </Alert>
) : (
    <InitBundlesTable initBundles={dataForFetch?.response?.items ?? []} />
);
```

Memoize the request function with `useCallback` when it closes over state the caller expects to drive refetches, so the hook's effect dependency stays stable.

### `useRestMutation` (POST, PUT, PATCH, DELETE)

Returns `{ mutate, reset, data, isIdle, isLoading, isSuccess, isError, status, error }`. Accepts global `onSuccess` / `onError` / `onSettled` callbacks in the hook options, and a local override on each `mutate` call. Canonical example from `Containers/Vulnerabilities/WorkloadCves/WatchedImages/UnwatchImageModal.tsx`:

```tsx
import useRestMutation from 'hooks/useRestMutation';
import { unwatchImage } from 'services/imageService';

const unwatchImageMutation = useRestMutation(
    (name: string) => unwatchImage(name),
    { onSuccess: onWatchedImagesChange }
);

<Button
    isDisabled={unwatchImageMutation.isLoading || unwatchImageMutation.isSuccess}
    isLoading={unwatchImageMutation.isLoading}
    onClick={() => unwatchImageMutation.mutate(unwatchImageName)}
>
    Unwatch image
</Button>;
```

When a mutation invalidates data read by a sibling `useRestQuery`, pair the two by passing the `refetch` from the query into the mutation's `onSuccess`:

```tsx
const { data, refetch } = useRestQuery(fetchClusterInitBundles);
const revokeMutation = useRestMutation(revokeInitBundle, {
    onSuccess: () => refetch(),
});
```

Call `reset()` when closing a modal so the next open starts from `isIdle`.

### `usePaginatedQuery`

Use this hook for infinite-load lists where the UI is a single scrollable surface with a "View more" action, not a traditional paged table. It accepts `(queryFn, pageSize, { dedupKeyFn, debounceRate, onError, manualFetch })` and returns `{ data: Item[][], page, fetchNextPage, resetPages, clearPages, isEndOfResults, isFetchingNextPage, isRefreshingResults, lastFetchError }`.

Key contract points:

- `data` is an `Item[][]` where each inner array is one fetched page. Flatten only in the component render layer.
- `dedupKeyFn` is how the hook survives source data changing faster than the UI loads. Provide it whenever the backing list can mutate between page fetches; omit it for stable catalogues.
- `fetchNextPage(immediate)` is debounced by default. Pass `true` for explicit user actions like a "View more" button click that is immediately disabled.
- `resetPages()` clears all cached pages and refetches from page 0. Call it when filter inputs change.
- `clearPages()` clears without refetching. Use this when unmounting a view whose state you do not want to persist.

### Cancellation

Use `makeCancellableAxiosRequest` from `services/cancellationUtils` when the request is heavy, when the user can trigger it rapidly (filter changes, search-as-you-type), or when a component may unmount mid-flight. The helper wraps an axios call with an `AbortController`, attaches the signal, and rejects with a `CancelledPromiseError` on abort:

```ts
import { makeCancellableAxiosRequest } from 'services/cancellationUtils';
import axios from './instance';

export function fetchHeavyReport(filter: FilterQuery) {
    return makeCancellableAxiosRequest((signal) =>
        axios
            .get<ReportResponse>('/v1/reports', { params: filter, signal })
            .then((response) => response.data)
    );
}
```

`useRestQuery` detects the `{ request, cancel }` shape via `isCancellableRequest` and invokes `cancel` on unmount or before a new request. In error handling, branch on `CancelledPromiseError` so aborted requests do not surface to the user as a failure:

```ts
import { CancelledPromiseError } from 'services/cancellationUtils';

if (error && !(error instanceof CancelledPromiseError)) {
    showAlert(getAxiosErrorMessage(error));
}
```

### Search, pagination, and query-string building

Use `searchOptionsToQuery` from `services/searchOptionsToQuery` to translate an array of `RestSearchOption` into the server's query string format, then `qs.stringify` to serialize the final URL. Use the `Pagination` and `FilterQuery` types from `services/types.ts` for paged endpoints. The `fetchClusters` example above shows the full pairing: `searchOptionsToQuery(options)` produces the `query` value, and `qs.stringify({ query }, { addQueryPrefix: true, arrayFormat: 'repeat', allowDots: true })` produces the final `?query=...` suffix that the server parses correctly. Do not hand-roll `encodeURIComponent` or inline `queryString.stringify` calls for filters. The server expects the `+`-joined `searchOptionsToQuery` output, and rolling your own will parse wrong.

### Error handling

Format every user-facing error with `getAxiosErrorMessage` from `utils/responseErrorUtils`. The helper handles four cases in order: `ApolloError` with a network result, axios errors with a 403 (returns a role-permissions message), axios errors with a `response.data.message` from the server, and network errors with no response. The standard rendering pattern is a `PatternFly` `Alert variant="warning"` inlined in the page body, as in `InitBundlesPage.tsx`:

```tsx
<Alert variant="warning" title="Unable to fetch cluster init bundles" component="p" isInline>
    {getAxiosErrorMessage(errorForFetch)}
</Alert>
```

Use `variant="danger"` only for blocking, destructive-action failures. Warnings are the default for page-level fetch errors because they do not imply the UI is broken.

### Types

Prefer request and response types generated from proto definitions under `types/*.proto.ts`. For no-op responses (delete endpoints, acknowledgements), import `Empty` from `services/types.ts` rather than defining a local `{}`. When no proto type exists, declare the shape in the service file itself next to the function that uses it, and export it alongside the function so callers share one source of truth. Do not invent response shapes in component files.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll just use GraphQL, it's the primary layer" | REST is the primary layer. GraphQL is scoped to existing GraphQL code and new backend-authored GraphQL endpoints. Defaulting to GraphQL for new features fights the codebase. |
| "I'll import `axios` directly from the package" | `services/instance.js` is the single permitted import site for the raw `axios` module. Every other file uses the instance so timeouts, interceptors, and future auth wiring stay in one place. |
| "I'll call `axios` directly from the component, it's faster" | Service functions are the seam between UI and server. Bypassing them breaks test mocking, cancellation support, and the ability to reuse the same call from two components. |
| "I'll use `useEffect` plus `useState` instead of `useRestQuery`" | `useRestQuery` already handles mount, unmount, cancellation, error clearing, and refetch. Rolling your own will leak state on unmount and miss at least one of these. |
| "Cancellation is overkill for this query" | Search-as-you-type, filter bars, and heavy reports all produce rapid sequences of requests. Without `makeCancellableAxiosRequest` a stale response can overwrite a fresh one. |
| "I'll stringify the query myself, it's just a few fields" | The server parses the `+`-joined output of `searchOptionsToQuery` plus `qs.stringify` with `arrayFormat: 'repeat'`. Custom string-building works until a filter contains a comma and then silently returns the wrong results. |

## Red Flags

- A component file that imports `axios` directly from the `axios` package.
- A REST call written inline in a component instead of delegated to a `services/` function.
- A `useRestQuery` call with only a data branch and no handling for `isLoading`, `error`, or the empty case.
- `error.message` rendered raw in an `Alert` instead of `getAxiosErrorMessage(error)`.
- Inline `queryString.stringify` or hand-built `?foo=bar&baz=` URLs for filter state.
- A new `gql` query or mutation added to a feature when the backend has not shipped a matching new GraphQL endpoint.

## Verification

After adding or modifying a REST call, confirm:

- [ ] The service function lives in `services/<Entity>Service.ts` and imports `axios` from `./instance`, not from the `axios` package.
- [ ] The component consumes the service through `useRestQuery`, `useRestMutation`, or `usePaginatedQuery`, not a direct `axios` call.
- [ ] Loading, error, and empty states are each rendered with an explicit branch.
- [ ] Errors are formatted via `getAxiosErrorMessage` and rendered in an `Alert` with the appropriate variant.
- [ ] Heavy or rapidly-changing queries use `makeCancellableAxiosRequest`, and the error branch filters out `CancelledPromiseError`.
- [ ] Search filters are produced by `searchOptionsToQuery`; query strings are built with `qs.stringify`.
- [ ] Request and response types come from `types/*.proto.ts` when a proto exists, and `Empty` from `services/types.ts` is used for empty responses.
- [ ] No new GraphQL queries or mutations were added unless editing existing GraphQL code or the backend is shipping a new GraphQL endpoint for this feature.
