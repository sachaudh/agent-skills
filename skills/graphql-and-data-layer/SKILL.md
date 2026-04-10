---
name: graphql-and-data-layer
description: Implements Apollo Client 3.8 patterns for GraphQL data fetching in StackRox UI. Use when writing queries, mutations, or managing cache, when handling loading/error states, when deciding between GraphQL and REST, or when debugging Apollo cache behavior.
---

# GraphQL and Data Layer

## Overview

StackRox UI uses Apollo Client 3.8 as the primary data layer with GraphQL. REST (via Axios) exists as a fallback for endpoints that have not been migrated. This skill covers typed query/mutation patterns, cache management, optimistic updates, error handling, and the decision framework for when to use GraphQL vs. REST.

## When to Use

- Writing a new GraphQL query or mutation
- Managing Apollo cache (reads, writes, evictions)
- Implementing optimistic updates for responsive UI
- Handling loading and error states for data fetches
- Deciding whether to use GraphQL or REST for a new endpoint
- Debugging stale data or unexpected cache behavior
- NOT for state management decisions (see `react-state-patterns`)
- NOT for E2E testing of GraphQL (see `cypress-e2e-testing`)

## GraphQL vs. REST Decision

```
Is there a GraphQL endpoint for this data?
  YES --> Use Apollo Client (GraphQL)
  NO  --> Is the backend team adding one?
    YES --> Wait for it, or build the UI with a temporary REST call
    NO  --> Use Axios (REST) via a service function in services/
```

**Default to GraphQL** for all new features. Use REST only when:
- The endpoint is REST-only with no GraphQL migration planned
- The operation is a file upload (multipart/form-data)
- The operation is a streaming/WebSocket endpoint

## Typed Query and Mutation Patterns

### Query Definition

Co-locate queries with the components that use them, or place them in `queries/`:

```tsx
// queries/vulnerabilities.ts
import { gql, TypedDocumentNode } from '@apollo/client';

export interface VulnerabilitiesData {
  vulnerabilities: {
    id: string;
    cve: string;
    severity: 'CRITICAL' | 'HIGH' | 'MEDIUM' | 'LOW';
    fixable: boolean;
    component: string;
    version: string;
  }[];
}

export interface VulnerabilitiesVars {
  query: string;
  severity?: string;
  pagination: {
    offset: number;
    limit: number;
  };
}

export const GET_VULNERABILITIES: TypedDocumentNode<
  VulnerabilitiesData,
  VulnerabilitiesVars
> = gql`
  query GetVulnerabilities($query: String!, $severity: String, $pagination: Pagination!) {
    vulnerabilities(query: $query, severity: $severity, pagination: $pagination) {
      id
      cve
      severity
      fixable
      component
      version
    }
  }
`;
```

### Using Queries in Components

```tsx
import { useQuery } from '@apollo/client';
import { GET_VULNERABILITIES } from '../queries/vulnerabilities';

export function VulnerabilityList({ searchQuery, severity }: VulnerabilityListProps) {
  const { data, loading, error, refetch } = useQuery(GET_VULNERABILITIES, {
    variables: {
      query: searchQuery,
      severity,
      pagination: { offset: 0, limit: 25 },
    },
    fetchPolicy: 'cache-and-network',
  });

  if (loading && !data) return <Bullseye><Spinner /></Bullseye>;
  if (error) return <ErrorState error={error} retry={refetch} />;
  if (!data?.vulnerabilities.length) return <EmptyState variant="no-results" />;

  return <VulnerabilityTable rows={data.vulnerabilities} />;
}
```

### Mutation with Error Handling

```tsx
import { useMutation } from '@apollo/client';

const SUPPRESS_VULNERABILITY: TypedDocumentNode<
  { suppressVulnerability: { id: string; suppressed: boolean } },
  { id: string; reason: string }
> = gql`
  mutation SuppressVulnerability($id: ID!, $reason: String!) {
    suppressVulnerability(id: $id, reason: $reason) {
      id
      suppressed
    }
  }
`;

export function useSuppressVulnerability() {
  const [suppress, { loading }] = useMutation(SUPPRESS_VULNERABILITY, {
    refetchQueries: ['GetVulnerabilities'],
    onError: (error) => {
      addAlert({ title: 'Failed to suppress vulnerability', variant: 'danger' });
      console.error('Suppress mutation failed:', error);
    },
  });

  return { suppress, loading };
}
```

## Cache Policies

Choose the right fetch policy for each query:

| Policy | When to Use |
|--------|------------|
| `cache-first` (default) | Stable reference data (CVE metadata, configuration schemas) |
| `cache-and-network` | Lists and dashboards that should show cached data immediately, then refresh |
| `network-only` | Data that must always be fresh (alerts, compliance scan results) |
| `no-cache` | One-off queries where caching adds no value (search suggestions) |

### Cache Key Configuration

```tsx
// apollo-client.ts
const cache = new InMemoryCache({
  typePolicies: {
    Vulnerability: {
      keyFields: ['id'],
    },
    Deployment: {
      keyFields: ['id', 'cluster', ['id']],
    },
    Query: {
      fields: {
        vulnerabilities: {
          // Merge paginated results
          keyArgs: ['query', 'severity'],
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          },
        },
      },
    },
  },
});
```

## Optimistic Updates

Use optimistic updates for actions where the expected outcome is predictable:

```tsx
const [toggleStar, { loading }] = useMutation(TOGGLE_STAR_DEPLOYMENT, {
  optimisticResponse: ({ id, starred }) => ({
    toggleStarDeployment: {
      __typename: 'Deployment',
      id,
      starred: !starred,
    },
  }),
  update(cache, { data }) {
    if (!data) return;
    cache.modify({
      id: cache.identify(data.toggleStarDeployment),
      fields: {
        starred: () => data.toggleStarDeployment.starred,
      },
    });
  },
  onError: (_error, _vars, restoreCache) => {
    // Apollo automatically rolls back optimistic response on error
    addAlert({ title: 'Failed to update star status', variant: 'danger' });
  },
});
```

### When NOT to Use Optimistic Updates

- The mutation's outcome depends on server-side validation
- The mutation affects multiple entities in unpredictable ways
- The mutation triggers server-side side effects that change other data

## Error and Loading States

Every query must handle three states:

```tsx
// Standard three-state pattern
export function DataComponent() {
  const { data, loading, error, refetch } = useQuery(QUERY);

  // 1. Loading: show skeleton or spinner (prefer skeleton for content areas)
  if (loading && !data) {
    return <DataComponentSkeleton />;
  }

  // 2. Error: show actionable error with retry
  if (error) {
    return (
      <EmptyState variant="danger">
        <EmptyStateHeader titleText="Unable to load data" />
        <EmptyStateBody>{error.message}</EmptyStateBody>
        <EmptyStateFooter>
          <Button variant="primary" onClick={() => refetch()}>Retry</Button>
        </EmptyStateFooter>
      </EmptyState>
    );
  }

  // 3. Empty: show meaningful empty state
  if (!data?.items.length) {
    return (
      <EmptyState>
        <EmptyStateHeader titleText="No results found" />
        <EmptyStateBody>Adjust your filters or search terms.</EmptyStateBody>
      </EmptyState>
    );
  }

  // 4. Data: render content
  return <DataTable rows={data.items} />;
}
```

### Background Refresh Pattern

When using `cache-and-network`, show stale data with a refresh indicator:

```tsx
const { data, loading, previousData } = useQuery(QUERY, {
  fetchPolicy: 'cache-and-network',
});

const displayData = data ?? previousData;
const isRefreshing = loading && !!displayData;

return (
  <>
    {isRefreshing && <ProgressBar aria-label="Refreshing data" />}
    {displayData && <DataTable rows={displayData.items} />}
  </>
);
```

## Service Layer for REST Fallback

When GraphQL is not available, use the service layer:

```tsx
// services/clusterService.ts
import axios from 'axios';
import { Cluster } from '../types/cluster';

const clustersUrl = '/v1/clusters';

export function fetchClusters(): Promise<Cluster[]> {
  return axios.get<{ clusters: Cluster[] }>(clustersUrl)
    .then((response) => response.data.clusters);
}

export function deleteCluster(id: string): Promise<void> {
  return axios.delete(`${clustersUrl}/${id}`);
}
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll skip TypedDocumentNode, the query is simple" | Untyped queries let variable and response shape errors through silently. TypedDocumentNode catches them at compile time. |
| "cache-first is fine for this list page" | List pages show data that changes frequently. Use `cache-and-network` so users see fresh data after mutations. |
| "I'll handle errors in the parent component" | Each query should handle its own error state. Parent-level error boundaries are a safety net, not a replacement. |
| "Optimistic updates are overkill for this mutation" | If the mutation is predictable (toggle, delete) and the user expects instant feedback, optimistic updates are the right call. |
| "I'll use REST because it's simpler" | REST skips the Apollo cache, which means no automatic UI updates, no optimistic responses, and manual state management for loading/error. |

## Red Flags

- Queries without TypeScript types (raw `gql` strings with no `TypedDocumentNode`)
- Missing loading or error state handling in components that use `useQuery`
- Using `network-only` everywhere instead of appropriate cache policies
- Inline GraphQL strings inside component files (extract to queries/)
- REST calls for endpoints that have GraphQL equivalents
- Direct cache manipulation (`cache.writeQuery`) without understanding merge policies
- Mutations without `refetchQueries` or cache updates (stale UI after mutation)

## Verification

After implementing data layer code:

- [ ] Queries and mutations use `TypedDocumentNode` with explicit types
- [ ] Fetch policy is intentionally chosen (not relying on default for everything)
- [ ] Loading state shows skeleton or spinner
- [ ] Error state shows actionable message with retry option
- [ ] Empty state shows meaningful guidance
- [ ] Mutations update the cache or trigger refetch of affected queries
- [ ] Optimistic updates used for predictable, user-facing mutations
- [ ] GraphQL documents are in `queries/` or co-located, not inline
- [ ] REST fallback uses service layer in `services/`, not inline Axios calls
