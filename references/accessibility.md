# Angular Accessibility — CDK a11y Detailed Reference

This file covers **low-level accessibility utilities** from `@angular/cdk/a11y`. For **high-level headless accessible components** (Tabs, Accordion, Menu, Listbox, Select, Autocomplete, Grid, Tree, Toolbar, Menubar, Multiselect, Combobox), see [angular-aria.md](angular-aria.md).

## Table of Contents

- [ARIA Attribute Binding](#aria-attribute-binding)
- [FocusTrap](#focustrap)
- [FocusMonitor](#focusmonitor)
- [LiveAnnouncer](#liveannouncer)
- [AriaDescriber](#ariadescriber)
- [Keyboard Navigation Managers](#keyboard-navigation-managers)
- [InteractivityChecker](#interactivitychecker)
- [InputModalityDetector](#inputmodalitydetector)
- [HighContrastModeDetector](#highcontrastmodedetector)
- [Accessible Patterns](#accessible-patterns)

## ARIA Attribute Binding

ARIA attributes have no corresponding DOM properties — always use the `attr.` prefix:

```html
<!-- Dynamic ARIA binding -->
<button [attr.aria-label]="actionLabel()">
<button [attr.aria-expanded]="isExpanded()">
<input [attr.aria-invalid]="isInvalid() ? 'true' : null">
<div [attr.aria-activedescendant]="activeOptionId()">
<nav [attr.aria-label]="navLabel()">

<!-- Conditionally remove attribute (set to null) -->
<div [attr.role]="isDialog() ? 'dialog' : null">
<input [attr.aria-describedby]="hasError() ? 'error-msg' : null">
```

Common ARIA attributes and their patterns:

| Attribute | Purpose | Angular Pattern |
|-----------|---------|-----------------|
| `aria-label` | Accessible name for icon-only buttons | `aria-label="Close"` or `[attr.aria-label]="label()"` |
| `aria-labelledby` | Reference another element as label | `[attr.aria-labelledby]="titleId"` |
| `aria-describedby` | Additional description (managed by `AriaDescriber`) | `[attr.aria-describedby]="hintId()"` |
| `aria-expanded` | Toggle/disclosure state | `[attr.aria-expanded]="isOpen()"` |
| `aria-controls` | Which element this one controls | `[attr.aria-controls]="'panel-' + id()"` |
| `aria-activedescendant` | Virtual focus (used by `ActiveDescendantKeyManager`) | `[attr.aria-activedescendant]="activeId()"` |
| `aria-live` | Live region (managed by `LiveAnnouncer`/`CdkAriaLive`) | `aria-live="polite"` |
| `aria-modal` | Indicates modal dialog | `aria-modal="true"` |
| `aria-haspopup` | Indicates popup trigger | `aria-haspopup="listbox"` |
| `aria-busy` | Loading state | `[attr.aria-busy]="isLoading()"` |

## FocusTrap

Traps Tab key focus within a DOM element. Import `CdkTrapFocus` for standalone components or use `FocusTrapFactory` programmatically.

### CdkTrapFocus Directive

```typescript
import { CdkTrapFocus } from '@angular/cdk/a11y';

@Component({
  imports: [CdkTrapFocus],
  template: `
    <div cdkTrapFocus [cdkTrapFocusAutoCapture]="true">
      <input cdkFocusInitial placeholder="Gets focus first" />
      <button>Submit</button>
      <button (click)="close()">Cancel</button>
    </div>
  `,
})
```

| Input | Type | Description |
|-------|------|-------------|
| `cdkTrapFocus` | `boolean` | Whether the focus trap is active |
| `cdkTrapFocusAutoCapture` | `boolean` | Auto-move focus into trap on activation, restore on deactivation |

Use `cdkFocusInitial` on an element to mark it as the initial focus target.

### FocusTrapFactory (Programmatic)

```typescript
import { FocusTrapFactory, FocusTrap } from '@angular/cdk/a11y';

@Component({...})
export class DialogComponent {
  private readonly _focusTrapFactory = inject(FocusTrapFactory);
  private _dialogEl = viewChild.required<ElementRef>('dialog');
  private _focusTrap!: FocusTrap;

  constructor() {
    afterNextRender(() => {
      this._focusTrap = this._focusTrapFactory.create(this._dialogEl().nativeElement);
      this._focusTrap.focusInitialElementWhenReady();
    });
  }

  public close() {
    this._focusTrap.destroy();
  }
}
```

**FocusTrap instance methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `focusInitialElement()` | `boolean` | Focus element marked with `cdkFocusInitial`, or first tabbable |
| `focusInitialElementWhenReady()` | `Promise<boolean>` | Same, but waits for next microtask |
| `focusFirstTabbableElement()` | `boolean` | Focus first tabbable element |
| `focusLastTabbableElement()` | `boolean` | Focus last tabbable element |
| `hasAttached()` | `boolean` | Whether focus trap anchors are in the DOM |
| `destroy()` | `void` | Remove anchors and deactivate |

## FocusMonitor

Monitors focus on elements and classifies origin: `'mouse'`, `'keyboard'`, `'touch'`, `'program'`, or `null` (blur).

### CdkMonitorFocus Directive

```typescript
import { CdkMonitorFocus } from '@angular/cdk/a11y';

@Component({
  imports: [CdkMonitorFocus],
  template: `
    <!-- Monitor single element -->
    <button cdkMonitorElementFocus
            (cdkFocusChange)="onFocusChange($event)">
      Click me
    </button>

    <!-- Monitor element AND children -->
    <div cdkMonitorSubtreeFocus
         (cdkFocusChange)="onFocusChange($event)">
      <input />
      <button>Inside</button>
    </div>
  `,
})
```

Automatically applies CSS classes: `.cdk-focused`, `.cdk-mouse-focused`, `.cdk-keyboard-focused`, `.cdk-touch-focused`, `.cdk-program-focused`.

### FocusMonitor Service

```typescript
import { FocusMonitor, FocusOrigin } from '@angular/cdk/a11y';

@Component({...})
export class MyComponent {
  private readonly _focusMonitor = inject(FocusMonitor);
  private _buttonEl = viewChild.required<ElementRef>('btn');

  constructor() {
    afterNextRender(() => {
      this._focusMonitor.monitor(this._buttonEl())
        .pipe(takeUntilDestroyed())
        .subscribe((origin: FocusOrigin) => {
          // origin: 'mouse' | 'keyboard' | 'touch' | 'program' | null
        });
    });
  }

  // Programmatic focus with explicit origin
  public focusViaKeyboard() {
    this._focusMonitor.focusVia(this._buttonEl(), 'keyboard');
  }
}
```

| Method | Description |
|--------|-------------|
| `monitor(el, checkChildren?)` | Returns `Observable<FocusOrigin>`, adds CSS classes |
| `stopMonitoring(el)` | Stop monitoring and remove classes |
| `focusVia(el, origin, options?)` | Focus element with specified origin |

## LiveAnnouncer

Announces messages to screen readers via a visually hidden `aria-live` region.

### Service Usage

```typescript
import { LiveAnnouncer } from '@angular/cdk/a11y';

@Component({...})
export class CartComponent {
  private readonly _liveAnnouncer = inject(LiveAnnouncer);

  public addToCart(item: string) {
    // Polite announcement (default) — waits for current speech to finish
    this._liveAnnouncer.announce(`${item} added to cart`);
  }

  public showError(message: string) {
    // Assertive — interrupts current speech
    this._liveAnnouncer.announce(message, 'assertive');
  }

  public save() {
    // Auto-clear after 3 seconds
    this._liveAnnouncer.announce('Changes saved', 3000);
  }
}
```

| Method | Description |
|--------|-------------|
| `announce(message, politeness?)` | Announce with `'polite'` (default) or `'assertive'` |
| `announce(message, duration?, politeness?)` | Announce and auto-clear after duration (ms) |
| `clear()` | Clear current announcement |

### CdkAriaLive Directive

```html
<!-- Automatically announces when text content changes -->
<div [cdkAriaLive]="'polite'">{{ statusMessage() }}</div>
<div [cdkAriaLive]="'assertive'">{{ errorMessage() }}</div>
```

### Configuration

```typescript
import { LIVE_ANNOUNCER_DEFAULT_OPTIONS } from '@angular/cdk/a11y';

// In app.config.ts providers:
{
  provide: LIVE_ANNOUNCER_DEFAULT_OPTIONS,
  useValue: { politeness: 'polite', duration: 5000 },
}
```

## AriaDescriber

Manages `aria-describedby` relationships. Reuses shared description elements to avoid DOM duplication.

```typescript
import { AriaDescriber } from '@angular/cdk/a11y';

@Component({...})
export class TooltipDirective {
  private readonly _ariaDescriber = inject(AriaDescriber);
  private readonly _el = inject(ElementRef);

  public tooltipText = input.required<string>();

  constructor() {
    effect(() => {
      this._ariaDescriber.describe(this._el.nativeElement, this.tooltipText());
    });

    inject(DestroyRef).onDestroy(() => {
      this._ariaDescriber.removeDescription(this._el.nativeElement, this.tooltipText());
    });
  }
}
```

| Method | Description |
|--------|-------------|
| `describe(host, message, role?)` | Add `aria-describedby` to host, create hidden message element |
| `describe(host, element)` | Add `aria-describedby` referencing an existing element |
| `removeDescription(host, message, role?)` | Remove the `aria-describedby` reference |

Creates a visually hidden container in the DOM with shared description elements. Multiple hosts pointing to the same message text share one element.

## Keyboard Navigation Managers

Three classes for keyboard-navigable lists. All accept `QueryList<T>` or `T[]`.

### FocusKeyManager (Roving Tabindex)

Moves **DOM focus** to the active item. Items must implement `FocusableOption`:

```typescript
import { FocusKeyManager, FocusableOption } from '@angular/cdk/a11y';

interface FocusableOption {
  focus(origin?: FocusOrigin): void;
  disabled?: boolean;
  getLabel?(): string;  // For typeahead
}
```

```typescript
@Component({
  selector: 'app-menu',
  template: `
    <ul role="menu" tabindex="0" (keydown)="onKeydown($event)">
      <ng-content />
    </ul>
  `,
})
export class MenuComponent {
  public items = contentChildren(MenuItemComponent);
  private _keyManager!: FocusKeyManager<MenuItemComponent>;

  constructor() {
    afterNextRender(() => {
      this._keyManager = new FocusKeyManager(this.items())
        .withWrap()
        .withVerticalOrientation()
        .withHomeAndEnd()
        .withTypeAhead();
    });
  }

  public onKeydown(event: KeyboardEvent) {
    this._keyManager.onKeydown(event);
  }
}

@Component({
  selector: 'app-menu-item',
  template: `<li role="menuitem" tabindex="-1" #el><ng-content /></li>`,
})
export class MenuItemComponent implements FocusableOption {
  private _element = viewChild.required<ElementRef>('el');
  public disabled = false;

  public focus() { this._element().nativeElement.focus(); }
  public getLabel() { return this._element().nativeElement.textContent?.trim() ?? ''; }
}
```

### ActiveDescendantKeyManager (aria-activedescendant)

Does NOT move DOM focus. Sets/clears active styles on items. Items must implement `Highlightable`:

```typescript
import { ActiveDescendantKeyManager, Highlightable } from '@angular/cdk/a11y';

interface Highlightable {
  setActiveStyles(): void;
  setInactiveStyles(): void;
  disabled?: boolean;
  getLabel?(): string;
}
```

```typescript
@Component({
  selector: 'app-listbox',
  template: `
    <ul role="listbox" tabindex="0"
        [attr.aria-activedescendant]="keyManager?.activeItem?.id"
        (keydown)="keyManager?.onKeydown($event)"
        (focus)="onFocus()">
      <ng-content />
    </ul>
  `,
})
export class ListboxComponent {
  public options = contentChildren(ListboxOptionComponent);
  private _keyManager!: ActiveDescendantKeyManager<ListboxOptionComponent>;

  constructor() {
    afterNextRender(() => {
      this._keyManager = new ActiveDescendantKeyManager(this.options())
        .withWrap()
        .withVerticalOrientation()
        .withHomeAndEnd()
        .withTypeAhead();
    });
  }

  public onFocus() {
    if (!this._keyManager.activeItem) {
      this._keyManager.setFirstItemActive();
    }
  }
}

@Component({
  selector: 'app-listbox-option',
  template: `
    <li role="option" [id]="id"
        [class.active]="isActive"
        [attr.aria-selected]="isActive">
      <ng-content />
    </li>
  `,
})
export class ListboxOptionComponent implements Highlightable {
  private static _nextId = 0;
  public id = `listbox-option-${ListboxOptionComponent._nextId++}`;
  public disabled = false;
  public isActive = false;

  public setActiveStyles() { this.isActive = true; }
  public setInactiveStyles() { this.isActive = false; }
  public getLabel() { return inject(ElementRef).nativeElement.textContent?.trim() ?? ''; }
}
```

### ListKeyManager Configuration (shared by both)

| Method | Description |
|--------|-------------|
| `withWrap()` | Wrap from last to first and vice versa |
| `withVerticalOrientation()` | Navigate with Up/Down arrows |
| `withHorizontalOrientation('ltr'\|'rtl')` | Navigate with Left/Right arrows |
| `withHomeAndEnd()` | Home/End jump to first/last item |
| `withPageUpDown(enabled?, delta?)` | Page Up/Down skip by delta |
| `withTypeAhead(debounceMs?)` | Type characters to jump to matching item |
| `skipPredicate(fn)` | Custom skip logic (e.g., skip hidden items) |
| `withAllowedModifierKeys(keys)` | Allow navigation while modifier keys held |

| Property/Method | Description |
|-----------------|-------------|
| `activeItem` | Currently active item (or `null`) |
| `activeItemIndex` | Index of active item (or `null`) |
| `change` | `Subject<number>` emitting on active item change |
| `setActiveItem(index\|item)` | Set active item directly |
| `setFirstItemActive()` | Activate first enabled item |
| `setLastItemActive()` | Activate last enabled item |
| `setNextItemActive()` | Activate next enabled item |
| `setPreviousItemActive()` | Activate previous enabled item |
| `onKeydown(event)` | Handle keyboard event |

## InteractivityChecker

Checks whether elements are focusable, tabbable, visible, or disabled.

```typescript
import { InteractivityChecker } from '@angular/cdk/a11y';

@Component({...})
export class MyComponent {
  private readonly _checker = inject(InteractivityChecker);

  public checkElement(el: HTMLElement) {
    this._checker.isFocusable(el);    // Can receive focus via script
    this._checker.isTabbable(el);     // Can receive focus via Tab key
    this._checker.isVisible(el);      // Not display:none or hidden attribute
    this._checker.isDisabled(el);     // Has disabled attribute

    // Check focusability ignoring current visibility
    this._checker.isFocusable(el, { ignoreVisibility: true });
  }
}
```

## InputModalityDetector

Detects the most recent input modality (keyboard, mouse, or touch) across the page. Useful for showing focus indicators only on keyboard navigation.

```typescript
import { InputModalityDetector, InputModality } from '@angular/cdk/a11y';

@Component({...})
export class AppComponent {
  private readonly _detector = inject(InputModalityDetector);

  constructor() {
    this._detector.modalityDetected
      .pipe(takeUntilDestroyed())
      .subscribe((modality: InputModality) => {
        // 'keyboard' | 'mouse' | 'touch' | null
        document.body.classList.toggle('keyboard-user', modality === 'keyboard');
      });
  }

  // Check current modality synchronously
  public get isKeyboard() { return this._detector.mostRecentModality === 'keyboard'; }
}
```

Configure ignored keys (default ignores Alt, Ctrl, Meta, Shift):

```typescript
import { INPUT_MODALITY_DETECTOR_OPTIONS } from '@angular/cdk/a11y';

{ provide: INPUT_MODALITY_DETECTOR_OPTIONS, useValue: { ignoreKeys: [SHIFT] } }
```

## HighContrastModeDetector

Detects Windows High Contrast Mode (forced-colors).

```typescript
import { HighContrastModeDetector, HighContrastMode } from '@angular/cdk/a11y';

@Component({...})
export class MyComponent {
  private readonly _hcDetector = inject(HighContrastModeDetector);
  public isHighContrast = this._hcDetector.getHighContrastMode() !== HighContrastMode.NONE;
}
```

CSS classes automatically added to `<body>`:

| Class | Meaning |
|-------|---------|
| `cdk-high-contrast-active` | Any high contrast mode is active |
| `cdk-high-contrast-black-on-white` | Black-on-white theme |
| `cdk-high-contrast-white-on-black` | White-on-black theme |

```css
.cdk-high-contrast-active .icon-button {
  outline: solid 1px currentColor;
}
```

## Accessible Patterns

### Accessible Dialog

```typescript
@Component({
  selector: 'app-dialog',
  imports: [CdkTrapFocus],
  template: `
    <div class="backdrop" (click)="close()"></div>
    <div class="panel" role="dialog" aria-modal="true"
         [attr.aria-labelledby]="titleId"
         cdkTrapFocus [cdkTrapFocusAutoCapture]="true">
      <h2 [id]="titleId">{{ title() }}</h2>
      <ng-content />
      <button (click)="close()">Close</button>
    </div>
  `,
})
export class DialogComponent {
  public title = input.required<string>();
  public closed = output<void>();

  private readonly _liveAnnouncer = inject(LiveAnnouncer);
  private _previousFocus = document.activeElement as HTMLElement;
  private _titleId = `dialog-title-${crypto.randomUUID()}`;

  constructor() {
    this._liveAnnouncer.announce(`Dialog opened: ${this.title()}`);
  }

  public close() {
    this._liveAnnouncer.announce('Dialog closed');
    this._previousFocus?.focus();
    this.closed.emit();
  }
}
```

### Keyboard-Only Focus Ring

```typescript
@Component({
  imports: [CdkMonitorFocus],
  template: `
    <button cdkMonitorElementFocus
            (cdkFocusChange)="focusOrigin.set($event)"
            [class.focus-visible]="focusOrigin() === 'keyboard'">
      <ng-content />
    </button>
  `,
  styles: `
    button:focus { outline: none; }
    button.focus-visible { outline: 2px solid var(--focus-color); outline-offset: 2px; }
  `,
})
export class SmartButtonComponent {
  public focusOrigin = signal<FocusOrigin>(null);
}
```

### Route Change Focus Management

```typescript
@Component({
  selector: 'app-root',
  template: `
    <a class="sr-only focus:not-sr-only" href="#main">Skip to content</a>
    <app-header />
    <main id="main" tabindex="-1" #mainContent>
      <router-outlet />
    </main>
  `,
})
export class AppComponent {
  private readonly _router = inject(Router);
  private readonly _focusMonitor = inject(FocusMonitor);
  private _mainContent = viewChild.required<ElementRef>('mainContent');

  constructor() {
    this._router.events.pipe(
      filter(e => e instanceof NavigationEnd),
      takeUntilDestroyed(),
    ).subscribe(() => {
      this._focusMonitor.focusVia(this._mainContent(), 'program');
    });
  }
}
```

### Accessible Loading State

```typescript
@Component({
  template: `
    <div [attr.aria-busy]="isLoading()">
      @if (isLoading()) {
        <div role="status">
          <span class="sr-only">Loading...</span>
          <app-spinner />
        </div>
      } @else {
        <div>{{ content() }}</div>
      }
    </div>
  `,
})
export class ContentComponent {
  private readonly _liveAnnouncer = inject(LiveAnnouncer);
  public isLoading = signal(true);
  public content = signal('');

  public async load() {
    this.isLoading.set(true);
    this._liveAnnouncer.announce('Loading content');
    // ... fetch data
    this.isLoading.set(false);
    this._liveAnnouncer.announce('Content loaded');
  }
}
```

### Accessible Form with Error Announcements

```html
<label for="email">Email</label>
<input id="email" type="email"
  [attr.aria-invalid]="emailInvalid() ? 'true' : null"
  [attr.aria-describedby]="emailInvalid() ? 'email-error' : 'email-hint'"
  [attr.aria-required]="'true'" />
<span id="email-hint" class="sr-only">Enter your email address</span>
@if (emailInvalid()) {
  <span id="email-error" role="alert">Please enter a valid email address.</span>
}
```

### Combobox / Autocomplete

```typescript
@Component({
  template: `
    <input role="combobox"
           [attr.aria-expanded]="isOpen()"
           [attr.aria-activedescendant]="keyManager?.activeItem?.id"
           aria-haspopup="listbox"
           [attr.aria-controls]="listboxId"
           (keydown)="onKeydown($event)"
           (input)="onInput($event)" />
    @if (isOpen()) {
      <ul role="listbox" [id]="listboxId">
        @for (option of filteredOptions(); track option.id) {
          <app-option [value]="option" (selected)="select(option)" />
        }
      </ul>
    }
  `,
})
export class ComboboxComponent {
  // Uses ActiveDescendantKeyManager — see KeyManagers section above
}
```

### Live Region for Table Updates

```html
<table><!-- ... --></table>
<div [cdkAriaLive]="'polite'">{{ tableAnnouncement() }}</div>
```

```typescript
public onSort(column: string, direction: string) {
  this._tableAnnouncement.set(`Table sorted by ${column}, ${direction}`);
}

public onFilter(count: number) {
  this._tableAnnouncement.set(`Showing ${count} results`);
}
```

### Visually Hidden (Screen Reader Only)

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

Tailwind CSS provides this as the `sr-only` utility class.
