# StackRox UI Patterns

Reusable UI patterns for StackRox frontend features. Each pattern includes the PatternFly components involved, the expected file layout, and a TypeScript skeleton. Use alongside `stackrox-ui-conventions`, `patternfly-development`, and `react-state-patterns`.

## Table of Contents

- [Table Page](#table-page)
- [Detail Panel](#detail-panel)
- [Form Wizard](#form-wizard)
- [Filter Bar](#filter-bar)
- [Empty State](#empty-state)
- [Error Boundary](#error-boundary)

## Table Page

A list view with a page header, filter bar, paginated table, and row actions. Used for most "list of X" pages (clusters, policies, alerts, violations).

**PatternFly composition:** `Page`, `PageSection`, `Toolbar`, `Table` (from `@patternfly/react-table`), `Pagination`, `EmptyState`, `Spinner`.

**File layout:**

```
containers/MyFeature/
  MyFeatureListPage.tsx       -- Container: data fetching + layout
  MyFeatureTable.tsx          -- Presentational table
  MyFeatureToolbar.tsx        -- Filter bar + search
  MyFeatureRowActions.tsx     -- Kebab menu per row
  hooks/useMyFeatureList.ts   -- Query + pagination state
```

**Skeleton:**

```tsx
// MyFeatureListPage.tsx
export function MyFeatureListPage() {
  const { items, totalCount, loading, error, pagination, filters } = useMyFeatureList();

  return (
    <>
      <PageSection variant="light">
        <Title headingLevel="h1">My Feature</Title>
      </PageSection>
      <PageSection>
        <MyFeatureToolbar
          filters={filters.value}
          onFiltersChange={filters.setValue}
          totalCount={totalCount}
          pagination={pagination}
        />
        {error && <Alert variant="danger" title="Failed to load" isInline>{error.message}</Alert>}
        {loading && !items && <Bullseye><Spinner /></Bullseye>}
        {items && items.length === 0 && <MyFeatureEmptyState />}
        {items && items.length > 0 && (
          <MyFeatureTable items={items} />
        )}
        <Pagination
          itemCount={totalCount}
          page={pagination.page}
          perPage={pagination.perPage}
          onSetPage={(_, page) => pagination.setPage(page)}
          onPerPageSelect={(_, perPage) => pagination.setPerPage(perPage)}
          variant="bottom"
        />
      </PageSection>
    </>
  );
}
```

**Rules:**

- Container owns data fetching; table is pure presentational and unit-testable
- Pagination state lives in URL query params (use `useSearchParams`) for shareable links
- Row actions use a `Dropdown` with `variant="plain"` and a kebab toggle
- Loading and error states are always handled, never swallowed silently

## Detail Panel

A side drawer showing details for a selected item in a list. Used for drill-down without leaving the list view.

**PatternFly composition:** `Drawer`, `DrawerContent`, `DrawerContentBody`, `DrawerPanelContent`, `DrawerHead`, `Tabs`, `DescriptionList`.

**File layout:**

```
containers/MyFeature/
  MyFeatureListWithDetail.tsx  -- Drawer wrapper
  MyFeatureDetailPanel.tsx     -- Panel contents
  MyFeatureDetailTabs/
    OverviewTab.tsx
    EventsTab.tsx
    SettingsTab.tsx
```

**Skeleton:**

```tsx
export function MyFeatureListWithDetail() {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  const drawerRef = useRef<HTMLDivElement>(null);

  const panelContent = selectedId && (
    <DrawerPanelContent>
      <DrawerHead>
        <Title headingLevel="h2">Details</Title>
        <DrawerActions>
          <DrawerCloseButton onClick={() => setSelectedId(null)} />
        </DrawerActions>
      </DrawerHead>
      <MyFeatureDetailPanel id={selectedId} />
    </DrawerPanelContent>
  );

  return (
    <Drawer isExpanded={!!selectedId} onExpand={() => drawerRef.current?.focus()}>
      <DrawerContent panelContent={panelContent}>
        <DrawerContentBody>
          <MyFeatureTable onRowClick={(item) => setSelectedId(item.id)} />
        </DrawerContentBody>
      </DrawerContent>
    </Drawer>
  );
}
```

**Rules:**

- Drawer state should sync to URL (`?selected=<id>`) for deep linking and shareability
- Fetch detail data when the drawer opens, not when the list loads
- Close the drawer when the selected item is deleted or filtered out
- Focus returns to the triggering row when the drawer closes

## Form Wizard

A multi-step form for creating or editing a complex entity. Used for policy creation, integration setup, and any flow with validation that spans multiple pages.

**PatternFly composition:** `Wizard` (PF6 composable API), `WizardStep`, `Form`, `FormGroup`, `TextInput`, Formik + Yup.

**File layout:**

```
containers/MyFeature/MyFeatureWizard/
  MyFeatureWizard.tsx          -- Wizard container
  schema.ts                    -- Yup schema per step + composed schema
  initialValues.ts             -- Typed initial values
  steps/
    BasicInfoStep.tsx
    ConfigurationStep.tsx
    ReviewStep.tsx
  useMyFeatureWizardSubmit.ts  -- Mutation + side effects
```

**Skeleton:**

```tsx
export function MyFeatureWizard({ onCancel }: { onCancel: () => void }) {
  const submit = useMyFeatureWizardSubmit();

  return (
    <Formik
      initialValues={initialValues}
      validationSchema={wizardSchema}
      onSubmit={submit.execute}
    >
      {({ values, validateForm, isSubmitting }) => (
        <Wizard onClose={onCancel} onSave={() => validateForm().then(submit.execute)}>
          <WizardStep name="Basic info" id="basic-info">
            <BasicInfoStep />
          </WizardStep>
          <WizardStep name="Configuration" id="configuration" isDisabled={!values.name}>
            <ConfigurationStep />
          </WizardStep>
          <WizardStep name="Review" id="review" footer={{ nextButtonText: 'Create' }}>
            <ReviewStep values={values} isSubmitting={isSubmitting} />
          </WizardStep>
        </Wizard>
      )}
    </Formik>
  );
}
```

**Rules:**

- Yup schema is the source of truth for validation; do not duplicate rules in components
- Each step validates its own fields before advancing (use `validateField` or step-scoped schemas)
- Cancellation should prompt if the form is dirty
- On success, navigate to the detail page of the created entity; on failure, stay on the wizard with an inline `Alert`

## Filter Bar

A horizontal toolbar with search, filter chips, and bulk actions. Used on every list page.

**PatternFly composition:** `Toolbar`, `ToolbarContent`, `ToolbarItem`, `ToolbarFilter`, `SearchInput`, `Select`, `ChipGroup`.

**File layout:**

```
containers/MyFeature/
  MyFeatureToolbar.tsx         -- Toolbar wrapper
  filters/
    StatusFilter.tsx
    NamespaceFilter.tsx
```

**Skeleton:**

```tsx
type Filters = {
  search: string;
  status: string[];
  namespace: string[];
};

export function MyFeatureToolbar({
  filters,
  onFiltersChange,
  totalCount,
}: {
  filters: Filters;
  onFiltersChange: (next: Filters) => void;
  totalCount: number;
}) {
  return (
    <Toolbar clearAllFilters={() => onFiltersChange(emptyFilters)}>
      <ToolbarContent>
        <ToolbarItem>
          <SearchInput
            value={filters.search}
            onChange={(_, value) => onFiltersChange({ ...filters, search: value })}
            onClear={() => onFiltersChange({ ...filters, search: '' })}
            placeholder="Search by name"
          />
        </ToolbarItem>
        <ToolbarFilter
          chips={filters.status}
          deleteChip={(_, chip) =>
            onFiltersChange({ ...filters, status: filters.status.filter((s) => s !== chip) })
          }
          categoryName="Status"
        >
          <StatusFilter
            selected={filters.status}
            onChange={(status) => onFiltersChange({ ...filters, status })}
          />
        </ToolbarFilter>
        <ToolbarItem variant="pagination" align={{ default: 'alignRight' }}>
          {totalCount} results
        </ToolbarItem>
      </ToolbarContent>
    </Toolbar>
  );
}
```

**Rules:**

- Filters sync to URL query parameters so views are shareable
- Clearing all filters is always available as a single action
- Filter chips are removable individually
- The total result count is always visible near the filter bar

## Empty State

The UI shown when a list has no items or a filter returns zero results. Every list view must have one.

**PatternFly composition:** `EmptyState`, `EmptyStateHeader`, `EmptyStateBody`, `EmptyStateFooter`, `EmptyStateActions`.

**File layout:**

```
components/EmptyStates/
  MyFeatureEmptyState.tsx
  MyFeatureFilteredEmptyState.tsx
```

**Skeleton:**

```tsx
import { CubesIcon } from '@patternfly/react-icons';

export function MyFeatureEmptyState({ onCreate }: { onCreate: () => void }) {
  return (
    <EmptyState>
      <EmptyStateHeader
        titleText="No features yet"
        icon={<EmptyStateIcon icon={CubesIcon} />}
        headingLevel="h2"
      />
      <EmptyStateBody>
        Create your first feature to get started. Features help you organize how StackRox
        evaluates your cluster.
      </EmptyStateBody>
      <EmptyStateFooter>
        <EmptyStateActions>
          <Button variant="primary" onClick={onCreate}>Create feature</Button>
        </EmptyStateActions>
      </EmptyStateFooter>
    </EmptyState>
  );
}
```

**Rules:**

- Distinguish "no data ever" from "no results after filtering" -- they deserve different copy
- Always include a next action (create, clear filters, learn more)
- Icon should match the feature's mental model; prefer PatternFly icons over custom SVG
- Copy should be specific and action-oriented, not generic ("No data available")

## Error Boundary

A component that catches rendering errors in its subtree and shows a fallback UI instead of crashing the whole app.

**PatternFly composition:** `EmptyState`, `EmptyStateHeader`, `EmptyStateBody`, `Button`, `ExclamationCircleIcon`.

**File layout:**

```
components/ErrorBoundary/
  ErrorBoundary.tsx          -- Generic boundary
  RouteErrorBoundary.tsx     -- Boundary for route-level errors
```

**Skeleton:**

```tsx
import { Component, type ErrorInfo, type ReactNode } from 'react';
import { ExclamationCircleIcon } from '@patternfly/react-icons';

type Props = { fallback?: ReactNode; children: ReactNode };
type State = { error: Error | null };

export class ErrorBoundary extends Component<Props, State> {
  state: State = { error: null };

  static getDerivedStateFromError(error: Error): State {
    return { error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    // Log to monitoring -- never swallow
    reportError(error, { componentStack: info.componentStack });
  }

  reset = () => this.setState({ error: null });

  render() {
    if (this.state.error) {
      return this.props.fallback ?? (
        <EmptyState variant="lg">
          <EmptyStateHeader
            titleText="Something went wrong"
            icon={<EmptyStateIcon icon={ExclamationCircleIcon} />}
            headingLevel="h2"
          />
          <EmptyStateBody>
            An unexpected error occurred. Try reloading the page or returning to the dashboard.
          </EmptyStateBody>
          <EmptyStateFooter>
            <Button variant="primary" onClick={this.reset}>Try again</Button>
          </EmptyStateFooter>
        </EmptyState>
      );
    }

    return this.props.children;
  }
}
```

**Rules:**

- Wrap every top-level route in an error boundary so one broken page does not crash the shell
- Always report the error to monitoring; never swallow silently
- Provide a recovery action (reload, reset, navigate away)
- Error boundaries catch render errors, not async errors -- handle promise rejections explicitly
- Use a single reusable `ErrorBoundary` component, not bespoke implementations per feature
