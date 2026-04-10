# PatternFly 6 Checklist

Quick reference checklist for PatternFly 6 usage in StackRox UI. Use alongside the `patternfly-development` skill.

## Table of Contents

- [Component Selection](#component-selection)
- [Accessibility](#accessibility)
- [Theming (Dark / Light)](#theming-dark--light)
- [Responsive Layout](#responsive-layout)
- [Deprecated Components and Replacements](#deprecated-components-and-replacements)
- [Imports](#imports)
- [Common Anti-Patterns](#common-anti-patterns)

## Component Selection

Before writing custom HTML or CSS, check whether PatternFly already solves the problem.

### Layout

- [ ] Stacked vertical layout uses `Stack` + `StackItem`
- [ ] Side-by-side layout uses `Split` + `SplitItem`
- [ ] Flex layouts use `Flex` + `FlexItem` with PatternFly spacers (not raw CSS)
- [ ] Grid layouts use `Grid` + `GridItem` with responsive column spans
- [ ] Page structure uses `Page`, `PageSection`, `PageSidebar`, `Masthead`
- [ ] Card containers use `Card`, `CardTitle`, `CardBody`, `CardFooter`

### Data Display

- [ ] Tabular data uses `Table` from `@patternfly/react-table`
- [ ] Rich lists use `DataList` or `SimpleList`
- [ ] Key/value metadata uses `DescriptionList`
- [ ] Status uses `Label`, `Badge`, or `Alert` (not custom colored pills)
- [ ] Empty states use `EmptyState` + `EmptyStateBody` + optional `EmptyStateFooter`
- [ ] Loading states use `Spinner` or `Bullseye` + `Spinner`

### Input

- [ ] Form fields wrapped in `Form` + `FormGroup`
- [ ] Text inputs use `TextInput`
- [ ] Dropdown selects use the current PF6 `Select` API (not the deprecated one)
- [ ] Checkbox, Radio, Switch use PatternFly components
- [ ] Buttons use `Button` with the correct `variant` (`primary`, `secondary`, `danger`, `link`, `plain`)
- [ ] Dangerous actions use `variant="danger"` + confirmation `Modal`

### Feedback

- [ ] Toast notifications use `AlertGroup` + `Alert` with `isLiveRegion`
- [ ] Inline warnings use `Alert` with `variant="warning"`
- [ ] Confirm dialogs use `Modal` with `variant="small"` and appropriate actions
- [ ] Progress indicators use `Progress`, not custom bars

## Accessibility

- [ ] Icon-only `Button` has `aria-label`
- [ ] `Modal` has `aria-label` or `aria-labelledby` pointing at its title
- [ ] `Select` has an associated `FormGroup` label or `aria-label`
- [ ] `Table` has a caption or `aria-label` describing its purpose
- [ ] Every interactive component is reachable via Tab key
- [ ] Focus returns to the trigger element when a `Modal` or `Popover` closes
- [ ] `Alert` with `variant="danger"` has `aria-live="assertive"` implied by `isLiveRegion`
- [ ] Color is not the only indicator of status (add icon + text)
- [ ] See `references/accessibility-checklist.md` for WCAG-level checks

## Theming (Dark / Light)

StackRox UI ships both light and dark themes. PatternFly 6 manages theming through CSS custom properties and a `pf-v6-theme-dark` class on the root element.

- [ ] No hardcoded colors (`#fff`, `#000`, `rgb(...)`) in components or CSS
- [ ] Use PatternFly design tokens: `--pf-v6-global--Color--100`, `--pf-v6-global--BackgroundColor--100`, etc.
- [ ] No absolute background colors on custom wrappers -- let PF tokens handle it
- [ ] Icons use `currentColor` so they adapt to theme
- [ ] Test every new view in both light and dark mode before merging
- [ ] Custom charts use PatternFly chart color tokens, not Tailwind or ad-hoc palettes
- [ ] Images with transparent backgrounds verified in dark mode

## Responsive Layout

- [ ] Use PatternFly breakpoints (`default`, `sm`, `md`, `lg`, `xl`, `2xl`)
- [ ] Grid columns use responsive span props (`md={6} lg={4}`)
- [ ] Nav collapses to a hamburger on small viewports via `PageSidebar`
- [ ] Tables that overflow use `gridBreakPoint="grid-md"` or a horizontal scroll container
- [ ] Modals use `variant="medium"` or smaller on mobile (avoid `large` on touch)
- [ ] Touch targets are at least 44x44 pixels (PatternFly defaults handle this; verify custom components)

## Deprecated Components and Replacements

These PF4 / PF5 / early-PF6 APIs are deprecated and should be replaced. Check the deprecation list every time PatternFly is upgraded.

| Deprecated | Replacement | Notes |
|---|---|---|
| `Dropdown` (old composable) | `Dropdown` from `@patternfly/react-core` PF6 API | New API uses `toggle` render prop |
| `Select` (old multi-variant) | `Select` PF6 composable API | See `skills/patternfly-development` for migration |
| `OptionsMenu` | `Dropdown` with `isPlain` | `OptionsMenu` removed in PF6 |
| `ApplicationLauncher` | `Dropdown` with icon toggle | Behavior preserved via custom toggle |
| `ContextSelector` | `Dropdown` or `Select` depending on use | Choose based on whether selection is single or multi |
| `Wizard` (v4 API) | `Wizard` PF6 API | New API is composable with explicit `WizardStep` |
| `@patternfly/react-core/deprecated/*` | Current PF6 paths | Any import from `/deprecated` is a red flag |
| `EmptyStateIcon` | `EmptyStateHeader` with `icon` prop | Header now owns icon rendering |
| `PageHeader`, `PageHeaderTools` | `Masthead` + `MastheadMain`, `MastheadContent` | Reorganized in PF6 |
| `Nav` horizontal variant with custom CSS | `Nav` with `variant="horizontal"` | Built-in variant available |

When in doubt, check the PatternFly 6 release notes for the specific component.

## Imports

- [ ] Import from `@patternfly/react-core`, `@patternfly/react-table`, `@patternfly/react-icons` (never from internal paths)
- [ ] No imports from `@patternfly/react-core/deprecated/*` without an accompanying migration ticket
- [ ] No deep imports like `@patternfly/react-core/dist/esm/...`
- [ ] Icon imports use named imports: `import { ExclamationCircleIcon } from '@patternfly/react-icons'`
- [ ] Do not import PatternFly CSS files directly -- rely on the global stylesheet loaded at app entry
- [ ] Avoid pulling in `@patternfly/react-styles` unless applying a specific PF utility class

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Raw `<div>` with manual flex CSS | Breaks theming, accessibility, consistency | Use `Flex` + `FlexItem` |
| Custom modal with `<div>` overlay | No focus trap, not accessible | Use `Modal` |
| Custom dropdown built from scratch | No keyboard support, no ARIA | Use `Dropdown` or `Select` |
| Hardcoded `color: '#151515'` | Breaks dark mode | Use PF color tokens |
| `<button className="my-btn">` | Bypasses PF button styling and states | Use `Button` |
| `<table>` with manual headers | No sorting, pagination, or accessibility | Use PF `Table` |
| Custom spinner | Unnecessary; PF one is well-tested | Use `Spinner` |
| Inline `style={{ marginTop: 16 }}` | Magic numbers; breaks rhythm | Use PF spacer components or utility classes |
| Importing from `/deprecated` long-term | Breaks on next PF major | Migrate now and delete the import |
| `className="pf-v5-..."` | Stale version prefix | Update to `pf-v6-...` |
