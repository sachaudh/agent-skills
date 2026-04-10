---
name: patternfly-development
description: Guides PatternFly 6 component usage in StackRox UI. Use when selecting, composing, or customizing PatternFly components, implementing dark/light theming, ensuring accessibility compliance, or troubleshooting PatternFly rendering issues.
---

# PatternFly Development

## Overview

PatternFly 6 is the design system for StackRox UI. Every UI component should use PatternFly primitives as the foundation. This skill covers component selection, theming, accessibility, and the decision of when to wrap vs. use PatternFly components directly. Getting PatternFly usage right eliminates custom CSS, ensures accessibility, and keeps the UI visually consistent.

## When to Use

- Selecting a PatternFly component for a new feature
- Deciding whether to wrap a PatternFly component or use it directly
- Implementing dark mode / light mode theming
- Fixing accessibility issues in PatternFly-based UI
- Upgrading PatternFly components after a major version bump
- NOT for general React patterns (see `frontend-ui-engineering`)
- NOT for project structure decisions (see `stackrox-ui-conventions`)

## Component Selection Decision Tree

When you need a UI element, start here:

```
Does PatternFly have this component?
  YES --> Does the PF component meet requirements as-is?
    YES --> Use it directly (no wrapper)
    NO  --> Can you compose multiple PF components?
      YES --> Compose them (preferred over custom CSS)
      NO  --> Wrap the PF component with additional behavior
  NO  --> Does PatternFly have a layout or utility that gets close?
    YES --> Build on top of the PF primitive
    NO  --> Build a custom component using PF CSS variables and tokens
```

### Direct Usage (Preferred)

Use PatternFly components directly when they meet requirements without modification:

```tsx
import { Button } from '@patternfly/react-core';

// Direct usage -- no wrapper needed
<Button variant="primary" onClick={handleSave}>Save</Button>
```

### Composition (Next Best)

Combine multiple PatternFly components to build complex UI:

```tsx
import {
  Toolbar,
  ToolbarContent,
  ToolbarItem,
  SearchInput,
  Select,
} from '@patternfly/react-core';

// Compose PF components for a filter toolbar
export function VulnerabilityFilterBar({ onSearch, onSeverityChange }: FilterBarProps) {
  return (
    <Toolbar>
      <ToolbarContent>
        <ToolbarItem>
          <SearchInput
            placeholder="Search vulnerabilities"
            onChange={(_event, value) => onSearch(value)}
          />
        </ToolbarItem>
        <ToolbarItem>
          <SeveritySelect onChange={onSeverityChange} />
        </ToolbarItem>
      </ToolbarContent>
    </Toolbar>
  );
}
```

### Wrapping (When Necessary)

Wrap only when you need to add behavior PatternFly does not provide:

```tsx
import { Select, SelectOption, SelectList, MenuToggle } from '@patternfly/react-core';

/**
 * StackRox-specific select that integrates with URL search params
 * for shareable filter state.
 */
export function URLParamSelect({ paramName, options, label }: URLParamSelectProps) {
  const [searchParams, setSearchParams] = useSearchParams();
  const selected = searchParams.get(paramName) ?? '';
  const [isOpen, setIsOpen] = useState(false);

  function handleSelect(_event: React.MouseEvent, value: string) {
    setSearchParams((prev) => {
      prev.set(paramName, value);
      return prev;
    });
    setIsOpen(false);
  }

  return (
    <Select
      isOpen={isOpen}
      selected={selected}
      onSelect={handleSelect}
      onOpenChange={setIsOpen}
      toggle={(toggleRef) => (
        <MenuToggle ref={toggleRef} onClick={() => setIsOpen(!isOpen)}>
          {selected || label}
        </MenuToggle>
      )}
    >
      <SelectList>
        {options.map((opt) => (
          <SelectOption key={opt.value} value={opt.value}>
            {opt.label}
          </SelectOption>
        ))}
      </SelectList>
    </Select>
  );
}
```

## Dark / Light Theming

StackRox supports both dark and light themes. PatternFly handles most theming through CSS custom properties.

### Rules

1. **Never use hardcoded colors.** Use PatternFly CSS variables:
   ```css
   /* Correct */
   color: var(--pf-t--global--text--color--regular);
   background: var(--pf-t--global--background--color--primary--default);

   /* Wrong */
   color: #333333;
   background: white;
   ```

2. **Test both themes.** Every component must render correctly in both dark and light mode.

3. **Use PatternFly's theme classes** on the root element:
   ```tsx
   // The theme class is set at the app root
   <div className="pf-v6-theme-dark">
     <App />
   </div>
   ```

4. **Custom component styling** must reference PatternFly tokens:
   ```css
   .custom-severity-badge {
     background: var(--pf-t--global--color--status--danger--default);
     color: var(--pf-t--global--text--color--on-status--danger);
     border-radius: var(--pf-t--global--border--radius--small);
     padding: var(--pf-t--global--spacer--xs) var(--pf-t--global--spacer--sm);
   }
   ```

## Accessibility

PatternFly components are accessible by default, but misuse breaks accessibility.

### Required Practices

1. **Always pass `aria-label` to data tables:**
   ```tsx
   <Table aria-label="Vulnerability results">{/* ... */}</Table>
   ```

2. **Use `ouiaId` for testing hooks** (Cypress, Testing Library):
   ```tsx
   <Button ouiaId="save-vulnerability-btn" variant="primary">Save</Button>
   ```

3. **Keyboard navigation:** PatternFly components handle keyboard nav internally. Do not override `onKeyDown` unless adding custom behavior.

4. **Focus management in modals and wizards:**
   ```tsx
   <Modal
     isOpen={isOpen}
     onClose={handleClose}
     aria-label="Edit vulnerability"
   >
     {/* Focus auto-trapped by PatternFly */}
   </Modal>
   ```

5. **Do not hide content from screen readers** unless it is purely decorative:
   ```tsx
   // Decorative icon -- hide from SR
   <ShieldIcon aria-hidden="true" />

   // Meaningful icon -- label it
   <ExclamationTriangleIcon aria-label="Critical severity" />
   ```

## Common PatternFly Pitfalls

| Pitfall | Correct Approach |
|---------|-----------------|
| Importing from internal paths (`@patternfly/react-core/dist/...`) | Always import from the package root: `@patternfly/react-core` |
| Using deprecated PF4/PF5 components in new code | Check the PF6 migration guide; use the PF6 equivalent |
| Overriding PatternFly styles with `!important` | Use PatternFly CSS variables or the component's `className` prop for custom theming |
| Forgetting `aria-label` on icon-only buttons | Every button without visible text needs `aria-label` |
| Using raw `<table>` instead of PatternFly `Table` | PatternFly Table handles sorting, selection, and accessibility automatically |
| Hardcoding spacing values (px, rem) | Use PatternFly spacer variables: `var(--pf-t--global--spacer--md)` |
| Not handling the `onOpenChange` callback on Select/Dropdown | PF6 requires explicit open-state management via `onOpenChange` |
| Using `<a>` tags for navigation instead of React Router `<Link>` | Use PatternFly's `component` prop to render as React Router Link |

## PatternFly Imports

Keep imports organized and from the correct packages:

```tsx
// Core components
import { Button, Modal, Alert } from '@patternfly/react-core';

// Table components
import { Table, Thead, Tbody, Tr, Th, Td } from '@patternfly/react-table';

// Icons
import { CheckCircleIcon, ExclamationTriangleIcon } from '@patternfly/react-icons';

// Charts (if needed)
import { Chart, ChartBar, ChartThemeColor } from '@patternfly/react-charts';

// Topology (if needed)
import { Visualization, VisualizationProvider } from '@patternfly/react-topology';
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "PatternFly doesn't have what I need, I'll build it custom" | Check the full component gallery first. PF6 has 80+ components. Most "custom" needs are compositions of existing PF components. |
| "I'll override the PF styles to match the design" | If the design diverges from PatternFly, raise it with the team. Overriding PF styles creates maintenance debt on every PF upgrade. |
| "Accessibility labels are noise in the code" | They are required for screen readers and are used by Cypress tests via `ouiaId`. They are functional, not decorative. |
| "I'll add dark mode support later" | Using PF CSS variables from the start costs nothing. Retrofitting hardcoded colors costs hours. |
| "The PF component API is too complex, a custom component is simpler" | PF complexity exists to handle accessibility, keyboard nav, and theming. A simpler custom component will miss all three. |

## Red Flags

- Hardcoded color values (`#hex`, `rgb()`) in component styles
- Direct DOM manipulation instead of using PatternFly's state management (e.g., manual dropdown toggle)
- Missing `aria-label` on interactive components without visible text
- Imports from PatternFly internal/dist paths
- Custom modal or dropdown implementations when PatternFly provides them
- Components that look different in dark mode vs. light mode due to hardcoded colors
- Using deprecated PatternFly 4/5 components in new code

## Verification

After implementing with PatternFly:

- [ ] All components use PatternFly primitives (no custom HTML for things PF provides)
- [ ] Colors use CSS custom properties, not hardcoded values
- [ ] Component renders correctly in both dark and light themes
- [ ] All interactive elements have appropriate `aria-label` or visible text
- [ ] `ouiaId` attributes added to key interactive elements for test hooks
- [ ] Imports are from package roots, not internal paths
- [ ] No deprecated PF4/PF5 components used in new code
- [ ] Wrapping is justified (direct usage or composition was insufficient)
- [ ] See `references/patternfly-checklist.md` for the full checklist
