# Angular Aria — Headless Accessible Components Reference

`@angular/aria` provides headless, accessible directives implementing common WAI-ARIA patterns. They manage keyboard interactions, ARIA attributes, focus management, and screen reader support — you provide HTML structure, CSS styling, and business logic.

Install: `npm install @angular/aria`

## Table of Contents

- [Accordion](#accordion)
- [Tabs](#tabs)
- [Listbox](#listbox)
- [Select](#select)
- [Multiselect](#multiselect)
- [Autocomplete](#autocomplete)
- [Combobox (Primitive)](#combobox-primitive)
- [Menu](#menu)
- [Menubar](#menubar)
- [Toolbar](#toolbar)
- [Grid](#grid)
- [Tree](#tree)
- [Component Selection Guide](#component-selection-guide)

## Accordion

Expandable/collapsible content panels following the WAI-ARIA accordion pattern.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `AccordionGroup` | `ngAccordionGroup` | Container managing expansion state |
| `AccordionTrigger` | `ngAccordionTrigger` | Button that toggles a panel |
| `AccordionPanel` | `ngAccordionPanel` | Content container paired with a trigger |
| `AccordionContent` | `ngAccordionContent` | Template wrapper enabling lazy rendering |

### Key Inputs

- **`AccordionGroup`**: `[multiExpandable]="true|false"` — allow multiple open panels (default: `true`)
- **`AccordionTrigger`**: `panelId` — unique ID linking trigger to panel; `disabled` — disables trigger
- **`AccordionTrigger`**: `expanded()` — signal returning current expansion state

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down | Navigate between triggers |
| Home / End | Jump to first/last trigger |
| Space / Enter | Toggle panel expansion |

### ARIA Managed

`aria-expanded`, `aria-controls`, `role="heading"`, `aria-level`

### Example

```typescript
import { Component } from '@angular/core';
import {
  AccordionGroup, AccordionTrigger, AccordionPanel, AccordionContent,
} from '@angular/aria/accordion';

@Component({
  selector: 'app-faq',
  imports: [AccordionGroup, AccordionTrigger, AccordionPanel, AccordionContent],
  template: `
    <div ngAccordionGroup [multiExpandable]="false">
      @for (item of faqItems; track item.id) {
        <h3>
          <span ngAccordionTrigger [panelId]="item.id"
                #trigger="ngAccordionTrigger">
            {{ item.question }}
          </span>
        </h3>
        <div ngAccordionPanel [panelId]="item.id">
          <ng-template ngAccordionContent>
            <p>{{ item.answer }}</p>
          </ng-template>
        </div>
      }
    </div>
  `,
})
export class FaqComponent {
  faqItems = [
    { id: 'q1', question: 'What is Angular?', answer: 'A web framework.' },
    { id: 'q2', question: 'What are signals?', answer: 'Reactive primitives.' },
  ];
}
```

**Use for:** FAQs, progressive disclosure, collapsible sections. **Not for:** navigation menus (use Menu), tabbed interfaces (use Tabs).

---

## Tabs

Tabbed interface organizing content into panels activated by tab buttons.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Tabs` | `ngTabs` | Root container |
| `TabList` | `ngTabList` | Manages tab buttons |
| `Tab` | `ngTab` | Individual tab trigger |
| `TabPanel` | `ngTabPanel` | Content container for a tab |
| `TabContent` | `ngTabContent` | Template wrapper for lazy panel content |

### Key Inputs

- **`TabList`**: `selectionMode` — `'follow'` (activate on focus) or `'explicit'` (Enter/Space to activate); `selectedTab` — initially selected tab value; `orientation` — `'horizontal'` or `'vertical'`
- **`Tab`**: `value` — unique identifier; `disabled` — prevents activation
- **`TabPanel`**: `value` — matches tab value; `[preserveContent]="true"` — keep DOM when inactive

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Left/Right (horizontal) | Navigate tabs |
| Arrow Up/Down (vertical) | Navigate tabs |
| Home / End | Jump to first/last tab |
| Space / Enter | Activate tab (explicit mode) |

### ARIA Managed

`aria-selected`, `aria-controls`, `role="tablist"`, `role="tab"`, `role="tabpanel"`, `tabindex`, `inert`

### Example

```typescript
import { Component } from '@angular/core';
import { Tabs, TabList, Tab, TabPanel, TabContent } from '@angular/aria/tabs';

@Component({
  selector: 'app-settings',
  imports: [Tabs, TabList, Tab, TabPanel, TabContent],
  template: `
    <div ngTabs>
      <div ngTabList selectionMode="follow" selectedTab="general">
        <div ngTab value="general">General</div>
        <div ngTab value="security">Security</div>
        <div ngTab value="notifications">Notifications</div>
      </div>

      <div ngTabPanel [preserveContent]="true" value="general">
        <ng-template ngTabContent>
          <p>General settings content</p>
        </ng-template>
      </div>
      <div ngTabPanel [preserveContent]="true" value="security">
        <ng-template ngTabContent>
          <p>Security settings content</p>
        </ng-template>
      </div>
      <div ngTabPanel [preserveContent]="true" value="notifications">
        <ng-template ngTabContent>
          <p>Notification preferences</p>
        </ng-template>
      </div>
    </div>
  `,
})
export class SettingsComponent {}
```

**Use for:** settings panels, documentation sections, dashboard views. **Not for:** sequential workflows (use stepper), page navigation (use routing).

---

## Listbox

Single or multi-selection list with keyboard navigation and type-ahead.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Listbox` | `ngListbox` | Container managing selection and navigation |
| `Option` | `ngOption` | Individual selectable option |

### Key Inputs

**Listbox:**

| Input | Default | Description |
|-------|---------|-------------|
| `multi` | `false` | Enable multiple selection |
| `orientation` | `'vertical'` | `'vertical'` or `'horizontal'` |
| `wrap` | `true` | Focus wraps at list edges |
| `selectionMode` | `'follow'` | `'follow'` (select on focus) or `'explicit'` (Space/Enter to select) |
| `focusMode` | `'roving'` | `'roving'` (roving tabindex) or `'activedescendant'` |
| `softDisabled` | `true` | Disabled items remain focusable |
| `readonly` | `false` | Read-only mode |
| `typeaheadDelay` | `500` | Type-ahead search reset (ms) |

**Option:** `value` (required), `label`, `disabled`

### Signals

- **`Listbox`**: `values()` — currently selected values (two-way bindable)
- **`Option`**: `selected()`, `active()` — selection and focus state

### Methods

- `scrollActiveItemIntoView(options?)` — scroll focused item into viewport
- `gotoFirst()` — navigate to first item

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down (vertical) | Navigate options |
| Arrow Left/Right (horizontal) | Navigate options |
| Space / Enter | Select (explicit mode) |
| Type characters | Type-ahead jump to matching option |

### ARIA Managed

`role="listbox"`, `aria-selected`, `aria-label`, `aria-multiselectable`

### Example

```typescript
import { Component } from '@angular/core';
import { Listbox, Option } from '@angular/aria/listbox';

@Component({
  selector: 'app-color-picker',
  imports: [Listbox, Option],
  template: `
    <div ngListbox selectionMode="explicit" aria-label="Choose a color">
      @for (color of colors; track color) {
        <div ngOption [value]="color">{{ color }}</div>
      }
    </div>
  `,
})
export class ColorPickerComponent {
  colors = ['Red', 'Green', 'Blue', 'Yellow'];
}
```

---

## Select

Single-selection dropdown combining a readonly combobox with a listbox popup. Uses CDK Overlay for positioning.

### Directives

Uses `Combobox`, `ComboboxInput`, `ComboboxPopupContainer` from `@angular/aria/combobox` combined with `Listbox`, `Option` from `@angular/aria/listbox`.

### Key Pattern

The combobox is set to `readonly` — no text input, just open/close a popup listbox.

### Example

```typescript
import { afterRenderEffect, ChangeDetectionStrategy, Component, computed, viewChild, viewChildren } from '@angular/core';
import { Combobox, ComboboxInput, ComboboxPopupContainer } from '@angular/aria/combobox';
import { Listbox, Option } from '@angular/aria/listbox';
import { OverlayModule } from '@angular/cdk/overlay';

@Component({
  selector: 'app-label-select',
  imports: [Combobox, ComboboxInput, ComboboxPopupContainer, Listbox, Option, OverlayModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div ngCombobox readonly>
      <div #origin class="select-trigger">
        <span>{{ displayValue() }}</span>
        <input ngComboboxInput aria-label="Select a label" />
        <span aria-hidden="true">arrow_drop_down</span>
      </div>
      <ng-template ngComboboxPopupContainer>
        <ng-template
          [cdkConnectedOverlay]="{origin, usePopover: 'inline', matchWidth: true}"
          [cdkConnectedOverlayOpen]="true">
          <div class="popup">
            <div ngListbox>
              @for (label of labels; track label) {
                <div ngOption [value]="label" [label]="label">
                  {{ label }}
                  <span aria-hidden="true">check</span>
                </div>
              }
            </div>
          </div>
        </ng-template>
      </ng-template>
    </div>
  `,
})
export class LabelSelectComponent {
  listbox = viewChild<Listbox<string>>(Listbox);
  options = viewChildren<Option<string>>(Option);
  combobox = viewChild<Combobox<string>>(Combobox);

  labels = ['Important', 'Starred', 'Work', 'Personal'];

  displayValue = computed(() => {
    const values = this.listbox()?.values() || [];
    return values.length ? values[0] : 'Select a label';
  });

  constructor() {
    // Scroll active option into view
    afterRenderEffect(() => {
      const option = this.options().find(opt => opt.active());
      setTimeout(() => option?.element.scrollIntoView({ block: 'nearest' }), 50);
    });

    // Reset scroll when closed
    afterRenderEffect(() => {
      if (!this.combobox()?.expanded()) {
        setTimeout(() => this.listbox()?.element.scrollTo(0, 0), 150);
      }
    });
  }
}
```

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down | Navigate options |
| Enter | Select option and close |
| Escape | Close without selecting |
| Space | Open/close popup |
| Home / End | Jump to first/last option |

**Use for:** fixed lists under 20 items. **Use Autocomplete for:** larger lists needing search. **Use radio buttons for:** 2-3 options.

---

## Multiselect

Multiple-selection dropdown combining a readonly combobox with a multi-enabled listbox.

### Pattern

Same directives as Select, but add `multi` to the listbox and `readonly` to the combobox.

### Example

```typescript
import { afterRenderEffect, ChangeDetectionStrategy, Component, computed, viewChild, viewChildren } from '@angular/core';
import { Combobox, ComboboxInput, ComboboxPopupContainer } from '@angular/aria/combobox';
import { Listbox, Option } from '@angular/aria/listbox';
import { OverlayModule } from '@angular/cdk/overlay';

@Component({
  selector: 'app-label-multiselect',
  imports: [Combobox, ComboboxInput, ComboboxPopupContainer, Listbox, Option, OverlayModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div ngCombobox readonly>
      <div #origin class="select-trigger">
        <span>{{ displayValue() }}</span>
        <input ngComboboxInput aria-label="Select labels" />
        <span aria-hidden="true">arrow_drop_down</span>
      </div>
      <ng-template ngComboboxPopupContainer>
        <ng-template
          [cdkConnectedOverlay]="{origin, usePopover: 'inline', matchWidth: true}"
          [cdkConnectedOverlayOpen]="true">
          <div class="popup">
            <div ngListbox multi>
              @for (label of labels; track label) {
                <div ngOption [value]="label" [label]="label">
                  <span>{{ label }}</span>
                  <span aria-hidden="true">check</span>
                </div>
              }
            </div>
          </div>
        </ng-template>
      </ng-template>
    </div>
  `,
})
export class LabelMultiselectComponent {
  listbox = viewChild<Listbox<string>>(Listbox);
  options = viewChildren<Option<string>>(Option);
  combobox = viewChild<Combobox<string>>(Combobox);

  labels = ['Important', 'Starred', 'Work', 'Personal', 'Travel'];

  displayValue = computed(() => {
    const values = this.listbox()?.values() || [];
    if (values.length === 0) return 'Select labels';
    if (values.length === 1) return values[0];
    return `${values[0]} + ${values.length - 1} more`;
  });

  constructor() {
    afterRenderEffect(() => {
      const option = this.options().find(opt => opt.active());
      setTimeout(() => option?.element.scrollIntoView({ block: 'nearest' }), 50);
    });
  }
}
```

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down | Navigate options |
| Space | Toggle option selection |
| Escape | Close dropdown |
| Enter | Confirm and close |

**Use for:** tags, categories, filters (under 20 options). **Not for:** single selection (use Select), large searchable lists (use Autocomplete), independent checkboxes.

---

## Autocomplete

Text input with filtered suggestion popup. Combines a combobox with text input and a filterable listbox.

### Directives

Uses `Combobox`, `ComboboxInput`, `ComboboxPopupContainer` from `@angular/aria/combobox` combined with `Listbox`, `Option` from `@angular/aria/listbox`.

### Filter Modes

- **`auto-select`** — automatically completes input to first match
- **`manual`** — user manually selects from filtered list
- **`highlight`** — highlights matching option without auto-selection

### Example

```typescript
import { afterRenderEffect, ChangeDetectionStrategy, Component, computed, signal, viewChild, viewChildren } from '@angular/core';
import { Combobox, ComboboxInput, ComboboxPopupContainer } from '@angular/aria/combobox';
import { Listbox, Option } from '@angular/aria/listbox';
import { OverlayModule } from '@angular/cdk/overlay';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-country-search',
  imports: [Combobox, ComboboxInput, ComboboxPopupContainer, Listbox, Option, OverlayModule, FormsModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div ngCombobox filterMode="auto-select">
      <div #origin class="autocomplete">
        <input [(ngModel)]="query" ngComboboxInput placeholder="Search countries" aria-label="Country" />
      </div>
      <ng-template ngComboboxPopupContainer>
        <ng-template
          [cdkConnectedOverlay]="{origin, usePopover: 'inline', matchWidth: true}"
          [cdkConnectedOverlayOpen]="true">
          <div class="popup">
            <div ngListbox>
              @for (country of filtered(); track country) {
                <div ngOption [value]="country">{{ country }}</div>
              }
            </div>
          </div>
        </ng-template>
      </ng-template>
    </div>
  `,
})
export class CountrySearchComponent {
  combobox = viewChild<Combobox<string>>(Combobox);
  listbox = viewChild<Listbox<string>>(Listbox);
  options = viewChildren<Option<string>>(Option);

  query = signal('');
  allCountries = ['Argentina', 'Australia', 'Austria', 'Belgium', 'Brazil', 'Canada'];

  filtered = computed(() =>
    this.allCountries.filter(c => c.toLowerCase().startsWith(this.query().toLowerCase()))
  );

  constructor() {
    afterRenderEffect(() => {
      const option = this.options().find(opt => opt.active());
      setTimeout(() => option?.element.scrollIntoView({ block: 'nearest' }), 50);
    });

    afterRenderEffect(() => {
      if (!this.combobox()?.expanded()) {
        setTimeout(() => this.listbox()?.element.scrollTo(0, 0), 150);
      }
    });
  }
}
```

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down | Navigate options |
| Enter | Select active option |
| Escape | Close popup |
| Typing | Filters options |

**Use for:** large lists needing search, country/city pickers. **Use Select for:** small fixed lists under 20 items.

---

## Combobox (Primitive)

The `Combobox`, `ComboboxInput`, and `ComboboxPopupContainer` directives from `@angular/aria/combobox` serve as the primitive foundation for Autocomplete, Select, and Multiselect patterns. Use them directly only when building custom implementations that don't fit the standard patterns.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Combobox` | `ngCombobox` | Coordinates text input with popup |
| `ComboboxInput` | `ngComboboxInput` | Marks the text input element |
| `ComboboxPopupContainer` | `ngComboboxPopupContainer` | Designates popup content area |

### Key Inputs

- **`Combobox`**: `filterMode` — `'auto-select'`, `'manual'`, or `'highlight'`; `readonly` — no text input
- **`Combobox`**: `expanded()` — signal tracking popup visibility

### ARIA Managed

`role="combobox"`, `aria-expanded`, `aria-selected`

### Integration with CDK Overlay

All combobox-based patterns use `@angular/cdk/overlay` for popup positioning:

```html
<ng-template
  [cdkConnectedOverlay]="{origin, usePopover: 'inline', matchWidth: true}"
  [cdkConnectedOverlayOpen]="true">
  <!-- popup content -->
</ng-template>
```

---

## Menu

Context menu or action menu triggered by a button, with optional nested submenus.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `MenuTrigger` | `ngMenuTrigger` | Button opening the menu |
| `Menu` | `ngMenu` | Menu container |
| `MenuContent` | `ngMenuContent` | Template wrapping menu items |
| `MenuItem` | `ngMenuItem` | Individual menu item |

### Key Inputs

- **`MenuTrigger`**: `[menu]` — reference to target menu
- **`MenuItem`**: `value` — item identifier; `[submenu]` — links to nested submenu; `disabled`
- **`Menu`**: generic typing `Menu<T>` for value types

### Signals

- `trigger.expanded()` — popup open state
- `item.visible()` — menu visibility for overlays
- `[data-active]` — attribute on currently focused item

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Up/Down | Navigate items |
| Home / End | Jump to first/last item |
| Enter / Space | Select item |
| Escape | Close menu |
| Arrow Right | Open submenu |
| Arrow Left | Close submenu |
| Type characters | Jump to matching item |

### ARIA Managed

`role="menu"`, `role="menuitem"`, `role="separator"`, `aria-expanded`, `aria-orientation`, `aria-hidden` (decorative icons)

### Example

```typescript
import { Component, viewChild } from '@angular/core';
import { Menu, MenuContent, MenuItem, MenuTrigger } from '@angular/aria/menu';
import { OverlayModule } from '@angular/cdk/overlay';

@Component({
  selector: 'app-actions-menu',
  imports: [Menu, MenuContent, MenuItem, MenuTrigger, OverlayModule],
  template: `
    <button ngMenuTrigger [menu]="actionsMenu()" #trigger="ngMenuTrigger">
      Actions
    </button>
    <ng-template
      [cdkConnectedOverlay]="{origin: trigger}"
      [cdkConnectedOverlayOpen]="trigger.expanded()"
      [cdkConnectedOverlayPositions]="[
        {originX: 'start', originY: 'bottom', overlayX: 'start', overlayY: 'top', offsetY: 4}
      ]">
      <div ngMenu #actionsMenu="ngMenu">
        <ng-template ngMenuContent>
          <div ngMenuItem value="edit">Edit</div>
          <div ngMenuItem value="duplicate">Duplicate</div>
          <div role="separator"></div>
          <div ngMenuItem value="delete">Delete</div>
        </ng-template>
      </div>
    </ng-template>
  `,
})
export class ActionsMenuComponent {
  actionsMenu = viewChild<Menu<string>>('actionsMenu');
}
```

**Use for:** context menus, action dropdowns. **Not for:** navigation (use Menubar), selection from options (use Select/Listbox).

---

## Menubar

Horizontal navigation bar with dropdown submenus, following the WAI-ARIA menubar pattern.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `MenuBar` | `ngMenuBar` | Root horizontal container |
| `Menu` | `ngMenu` | Submenu container |
| `MenuItem` | `ngMenuItem` | Menu item (top-level or nested) |
| `MenuContent` | `ngMenuContent` | Template wrapping submenu items |

### Key Inputs

Same as Menu, plus `MenuBar` as the root container.

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Left/Right | Navigate top-level items |
| Arrow Down | Open submenu / next item |
| Arrow Up | Previous item / close submenu |
| Enter / Space | Activate item |
| Escape | Close open menu |

### ARIA Managed

`role="menubar"`, `role="menu"`, `role="menuitem"`, `aria-expanded`, `aria-disabled`, `role="separator"`

### Example

```typescript
import { Component, viewChild } from '@angular/core';
import { MenuBar, Menu, MenuItem, MenuContent } from '@angular/aria/menu';
import { OverlayModule } from '@angular/cdk/overlay';

@Component({
  selector: 'app-main-menubar',
  imports: [MenuBar, Menu, MenuItem, MenuContent, OverlayModule],
  template: `
    <div ngMenuBar>
      <div ngMenuItem value="File" [submenu]="fileMenu()">File</div>
      <div ngMenu #fileMenu="ngMenu">
        <ng-template ngMenuContent>
          <div ngMenuItem value="New">New</div>
          <div ngMenuItem value="Open">Open</div>
          <div ngMenuItem value="Save">Save</div>
        </ng-template>
      </div>

      <div ngMenuItem value="Edit" [submenu]="editMenu()">Edit</div>
      <div ngMenu #editMenu="ngMenu">
        <ng-template ngMenuContent>
          <div ngMenuItem value="Undo">Undo</div>
          <div ngMenuItem value="Redo">Redo</div>
          <div role="separator"></div>
          <div ngMenuItem value="Cut">Cut</div>
          <div ngMenuItem value="Copy">Copy</div>
          <div ngMenuItem value="Paste">Paste</div>
        </ng-template>
      </div>
    </div>
  `,
})
export class MainMenubarComponent {
  fileMenu = viewChild<Menu<string>>('fileMenu');
  editMenu = viewChild<Menu<string>>('editMenu');
}
```

**Use for:** application-level navigation bars. **Not for:** simple action lists (use Menu), tabbed content (use Tabs).

---

## Toolbar

Groups related interactive controls (buttons, toggles) with single-tab-stop keyboard navigation.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Toolbar` | `ngToolbar` | Container with roving keyboard navigation |
| `ToolbarWidget` | `ngToolbarWidget` | Individual toolbar button/control |
| `ToolbarWidgetGroup` | `ngToolbarWidgetGroup` | Group of related controls (e.g., radio group) |

### Key Inputs

- **`Toolbar`**: `orientation` — `'horizontal'` (default) or `'vertical'`; `aria-label` (required)
- **`ToolbarWidget`**: `value` — identifier; `selected()` — signal for toggle/pressed state

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Left/Right (horizontal) | Navigate between widgets |
| Arrow Up/Down (vertical) | Navigate between widgets |
| Enter / Space | Activate widget |
| Tab | Move focus out of toolbar |

### ARIA Managed

`aria-pressed` (toggle buttons), `aria-checked` (radio buttons), `role="radio"`, `role="separator"`

### Example

```typescript
import { Component } from '@angular/core';
import { Toolbar, ToolbarWidget, ToolbarWidgetGroup } from '@angular/aria/toolbar';

@Component({
  selector: 'app-text-toolbar',
  imports: [Toolbar, ToolbarWidget, ToolbarWidgetGroup],
  template: `
    <div ngToolbar aria-label="Text Formatting">
      <button ngToolbarWidget value="bold" #bold="ngToolbarWidget"
              [aria-pressed]="bold.selected()" type="button">
        Bold
      </button>
      <button ngToolbarWidget value="italic" #italic="ngToolbarWidget"
              [aria-pressed]="italic.selected()" type="button">
        Italic
      </button>
      <button ngToolbarWidget value="underline" #underline="ngToolbarWidget"
              [aria-pressed]="underline.selected()" type="button">
        Underline
      </button>
    </div>
  `,
})
export class TextToolbarComponent {}
```

**Use for:** text editor toolbars, grouped action buttons. **Not for:** unrelated actions, deep nested navigation.

---

## Grid

Two-dimensional keyboard navigation for interactive data tables and calendar-like interfaces.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Grid` | `ngGrid` | Container on `<table>` element |
| `GridRow` | `ngGridRow` | Row-level navigation on `<tr>` |
| `GridCell` | `ngGridCell` | Cell-level navigation on `<td>` / `<th>` |
| `GridCellWidget` | `ngGridCellWidget` | Interactive element within a cell |

### Key Inputs

**Grid:**

| Input | Description |
|-------|-------------|
| `colWrap` | `'continuous'`, `'loop'`, or `'nowrap'` |
| `rowWrap` | `'continuous'`, `'loop'`, or `'nowrap'` |
| `enableSelection` | Enable cell selection |
| `selectionMode` | `'explicit'` or other modes |
| `softDisabled` | Disabled cells remain focusable |

**GridCell:** `rowSpan`, `colSpan`, `selected`, `disabled`

**GridCellWidget:** `widgetType` — e.g., `'editable'`

### Outputs

- `(activated)` — widget enters edit mode
- `(deactivated)` — widget exits edit mode

### Keyboard

| Key | Action |
|-----|--------|
| Arrow keys | Navigate between cells |
| Home / End | First/last cell in row |
| Page Up/Down | Navigate between rows |
| Enter | Activate cell widget / toggle selection |

### ARIA Managed

`role="grid"`, `role="row"`, `role="gridcell"`, `aria-selected`, `aria-disabled`, `aria-label`

### Example — Data Table

```typescript
import { Component, signal } from '@angular/core';
import { Grid, GridRow, GridCell, GridCellWidget } from '@angular/aria/grid';
import { FormsModule } from '@angular/forms';

interface Task {
  id: number;
  summary: string;
  priority: string;
  assignee: string;
}

@Component({
  selector: 'app-task-grid',
  imports: [Grid, GridRow, GridCell, GridCellWidget, FormsModule],
  template: `
    <table ngGrid>
      <thead>
        <tr ngGridRow>
          <th ngGridCell>ID</th>
          <th ngGridCell>Task</th>
          <th ngGridCell>Priority</th>
          <th ngGridCell>Assignee</th>
        </tr>
      </thead>
      <tbody>
        @for (task of tasks(); track task.id) {
          <tr ngGridRow>
            <td ngGridCell>{{ task.id }}</td>
            <td ngGridCell>{{ task.summary }}</td>
            <td ngGridCell>{{ task.priority }}</td>
            <td ngGridCell>
              <div ngGridCellWidget widgetType="editable" aria-label="edit assignee">
                <span>{{ task.assignee }}</span>
                <input [(ngModel)]="task.assignee" />
              </div>
            </td>
          </tr>
        }
      </tbody>
    </table>
  `,
})
export class TaskGridComponent {
  tasks = signal<Task[]>([
    { id: 101, summary: 'Build grid component', priority: 'High', assignee: 'Alice' },
    { id: 102, summary: 'Write documentation', priority: 'Medium', assignee: 'Bob' },
  ]);
}
```

### Example — Calendar

```typescript
import { Component, computed, signal } from '@angular/core';
import { Grid, GridRow, GridCell, GridCellWidget } from '@angular/aria/grid';

@Component({
  selector: 'app-calendar',
  imports: [Grid, GridRow, GridCell, GridCellWidget],
  template: `
    <table ngGrid colWrap="continuous" rowWrap="nowrap" [enableSelection]="true">
      <thead>
        <tr>
          @for (day of weekdays; track day) {
            <th scope="col">{{ day }}</th>
          }
        </tr>
      </thead>
      <tbody>
        @for (week of weeks(); track $index) {
          <tr ngGridRow>
            @for (day of week; track day.date) {
              <td ngGridCell [(selected)]="day.selected">
                <button ngGridCellWidget [attr.aria-label]="day.label">
                  {{ day.date }}
                </button>
              </td>
            }
          </tr>
        }
      </tbody>
    </table>
  `,
})
export class CalendarComponent {
  weekdays = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
  weeks = signal<{ date: number; label: string; selected: boolean }[][]>([]);
}
```

**Use for:** interactive data tables, calendars, spreadsheet UIs. **Not for:** read-only tables (use plain `<table>`), single-column lists (use Listbox), hierarchical data (use Tree).

---

## Tree

Hierarchical data display with expand/collapse and keyboard navigation.

### Directives

| Directive | Selector | Purpose |
|-----------|----------|---------|
| `Tree` | `ngTree` | Root tree container |
| `TreeItem` | `ngTreeItem` | Individual tree node |
| `TreeItemGroup` | `ngTreeItemGroup` | Groups child items under a parent |

### Key Inputs

**Tree:**

| Input | Default | Description |
|-------|---------|-------------|
| `values` | — | Two-way binding for selected values |
| `multi` | `false` | Enable multi-selection |
| `nav` | `false` | Navigation mode using `aria-current` |

**TreeItem:** `value`, `label`, `parent` (reference to parent tree/group), `disabled`, `expanded` (two-way), `selectable`

**TreeItemGroup:** `ownedBy` — reference to owning tree item

### Keyboard

| Key | Action |
|-----|--------|
| Arrow Down | Focus next item |
| Arrow Up | Focus previous item |
| Arrow Right | Expand parent or move to first child |
| Arrow Left | Collapse parent or move to parent |
| Home / End | Focus first/last item |
| Space | Toggle selection |
| Ctrl+A | Select all (multi-mode) |
| Type characters | Type-ahead jump |

### ARIA Managed

`role="tree"`, `role="group"`, `aria-expanded`, `aria-selected`, `aria-current` (nav mode), `aria-disabled`, `aria-label`

### Example

```typescript
import { Component, signal } from '@angular/core';
import { Tree, TreeItem, TreeItemGroup } from '@angular/aria/tree';
import { NgTemplateOutlet } from '@angular/common';

type TreeNode = {
  name: string;
  value: string;
  children?: TreeNode[];
  disabled?: boolean;
  expanded?: boolean;
};

@Component({
  selector: 'app-file-tree',
  imports: [Tree, TreeItem, TreeItemGroup, NgTemplateOutlet],
  template: `
    <ul ngTree #tree="ngTree" [(values)]="selected">
      <ng-template [ngTemplateOutlet]="nodes"
                   [ngTemplateOutletContext]="{ items: data, parent: tree }" />
    </ul>

    <ng-template #nodes let-items="items" let-parent="parent">
      @for (item of items; track item.value) {
        <li ngTreeItem
            [parent]="parent"
            [value]="item.value"
            [label]="item.name"
            [disabled]="item.disabled"
            [(expanded)]="item.expanded"
            #treeItem="ngTreeItem">
          {{ item.name }}
        </li>
        @if (item.children) {
          <ul role="group">
            <ng-template ngTreeItemGroup [ownedBy]="treeItem" #group="ngTreeItemGroup">
              <ng-template [ngTemplateOutlet]="nodes"
                           [ngTemplateOutletContext]="{ items: item.children, parent: group }" />
            </ng-template>
          </ul>
        }
      }
    </ng-template>
  `,
})
export class FileTreeComponent {
  selected = signal(['readme']);

  data: TreeNode[] = [
    {
      name: 'src', value: 'src', expanded: true,
      children: [
        {
          name: 'app', value: 'app', expanded: true,
          children: [
            { name: 'app.component.ts', value: 'app-comp' },
            { name: 'app.config.ts', value: 'app-config' },
          ],
        },
        { name: 'main.ts', value: 'main' },
      ],
    },
    { name: 'README.md', value: 'readme' },
    { name: 'package.json', value: 'pkg' },
  ];
}
```

**Use for:** file browsers, folder hierarchies, nested navigation. **Not for:** flat lists (use Listbox), data tables (use Grid), simple dropdowns (use Select).

---

## Component Selection Guide

| Need | Component | Package |
|------|-----------|---------|
| Collapsible content sections | Accordion | `@angular/aria/accordion` |
| Tabbed content panels | Tabs | `@angular/aria/tabs` |
| Single selection from list | Listbox | `@angular/aria/listbox` |
| Dropdown with fixed options (<20) | Select | `@angular/aria/combobox` + `listbox` |
| Dropdown with multiple selection | Multiselect | `@angular/aria/combobox` + `listbox` |
| Searchable dropdown (>20 options) | Autocomplete | `@angular/aria/combobox` + `listbox` |
| Context/action menu | Menu | `@angular/aria/menu` |
| App-level navigation bar | Menubar | `@angular/aria/menu` |
| Grouped action buttons | Toolbar | `@angular/aria/toolbar` |
| Interactive data table / calendar | Grid | `@angular/aria/grid` |
| Hierarchical data (files, folders) | Tree | `@angular/aria/tree` |

### When to use `@angular/aria` vs CDK a11y

- **`@angular/aria`** — Use for standard interactive UI patterns (tabs, menus, listboxes, etc.). These directives handle all ARIA attributes, keyboard interactions, and focus management automatically.
- **CDK a11y** — Use for lower-level utilities: `FocusTrap` (dialogs), `FocusMonitor` (focus-origin detection), `LiveAnnouncer` (screen reader messages), `AriaDescriber` (descriptions), keyboard navigation managers for custom widgets, `InteractivityChecker`, `InputModalityDetector`, `HighContrastModeDetector`.
- **Both together** — Angular Aria components use CDK Overlay for positioning popups. You may combine Angular Aria components with CDK a11y utilities (e.g., `LiveAnnouncer` for status announcements alongside a Tabs component).

### Key Patterns Across All Components

- All components are **headless** — bring your own styles
- All support **RTL** (right-to-left) languages automatically
- All use **Angular signals** for reactive state (`expanded()`, `selected()`, `active()`, `values()`)
- Popup-based components (Select, Multiselect, Autocomplete, Menu) integrate with **CDK Overlay** (`@angular/cdk/overlay`)
- Import directives directly into standalone components — no module import needed
