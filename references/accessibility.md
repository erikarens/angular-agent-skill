# Angular Accessibility — Detailed Reference

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
  private focusTrapFactory = inject(FocusTrapFactory);
  private dialogEl = viewChild.required<ElementRef>('dialog');
  private focusTrap!: FocusTrap;

  constructor() {
    afterNextRender(() => {
      this.focusTrap = this.focusTrapFactory.create(this.dialogEl().nativeElement);
      this.focusTrap.focusInitialElementWhenReady();
    });
  }

  close() {
    this.focusTrap.destroy();
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
  private focusMonitor = inject(FocusMonitor);
  private buttonEl = viewChild.required<ElementRef>('btn');

  constructor() {
    afterNextRender(() => {
      this.focusMonitor.monitor(this.buttonEl())
        .pipe(takeUntilDestroyed())
        .subscribe((origin: FocusOrigin) => {
          // origin: 'mouse' | 'keyboard' | 'touch' | 'program' | null
        });
    });
  }

  // Programmatic focus with explicit origin
  focusViaKeyboard() {
    this.focusMonitor.focusVia(this.buttonEl(), 'keyboard');
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
  private liveAnnouncer = inject(LiveAnnouncer);

  addToCart(item: string) {
    // Polite announcement (default) — waits for current speech to finish
    this.liveAnnouncer.announce(`${item} added to cart`);
  }

  showError(message: string) {
    // Assertive — interrupts current speech
    this.liveAnnouncer.announce(message, 'assertive');
  }

  save() {
    // Auto-clear after 3 seconds
    this.liveAnnouncer.announce('Changes saved', 3000);
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
  private ariaDescriber = inject(AriaDescriber);
  private el = inject(ElementRef);

  tooltipText = input.required<string>();

  constructor() {
    effect(() => {
      this.ariaDescriber.describe(this.el.nativeElement, this.tooltipText());
    });

    inject(DestroyRef).onDestroy(() => {
      this.ariaDescriber.removeDescription(this.el.nativeElement, this.tooltipText());
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
  items = contentChildren(MenuItemComponent);
  private keyManager!: FocusKeyManager<MenuItemComponent>;

  constructor() {
    afterNextRender(() => {
      this.keyManager = new FocusKeyManager(this.items())
        .withWrap()
        .withVerticalOrientation()
        .withHomeAndEnd()
        .withTypeAhead();
    });
  }

  onKeydown(event: KeyboardEvent) {
    this.keyManager.onKeydown(event);
  }
}

@Component({
  selector: 'app-menu-item',
  template: `<li role="menuitem" tabindex="-1" #el><ng-content /></li>`,
})
export class MenuItemComponent implements FocusableOption {
  private element = viewChild.required<ElementRef>('el');
  disabled = false;

  focus() { this.element().nativeElement.focus(); }
  getLabel() { return this.element().nativeElement.textContent?.trim() ?? ''; }
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
  options = contentChildren(ListboxOptionComponent);
  keyManager!: ActiveDescendantKeyManager<ListboxOptionComponent>;

  constructor() {
    afterNextRender(() => {
      this.keyManager = new ActiveDescendantKeyManager(this.options())
        .withWrap()
        .withVerticalOrientation()
        .withHomeAndEnd()
        .withTypeAhead();
    });
  }

  onFocus() {
    if (!this.keyManager.activeItem) {
      this.keyManager.setFirstItemActive();
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
  private static nextId = 0;
  id = `listbox-option-${ListboxOptionComponent.nextId++}`;
  disabled = false;
  isActive = false;

  setActiveStyles() { this.isActive = true; }
  setInactiveStyles() { this.isActive = false; }
  getLabel() { return inject(ElementRef).nativeElement.textContent?.trim() ?? ''; }
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
  private checker = inject(InteractivityChecker);

  checkElement(el: HTMLElement) {
    this.checker.isFocusable(el);    // Can receive focus via script
    this.checker.isTabbable(el);     // Can receive focus via Tab key
    this.checker.isVisible(el);      // Not display:none or hidden attribute
    this.checker.isDisabled(el);     // Has disabled attribute

    // Check focusability ignoring current visibility
    this.checker.isFocusable(el, { ignoreVisibility: true });
  }
}
```

## InputModalityDetector

Detects the most recent input modality (keyboard, mouse, or touch) across the page. Useful for showing focus indicators only on keyboard navigation.

```typescript
import { InputModalityDetector, InputModality } from '@angular/cdk/a11y';

@Component({...})
export class AppComponent {
  private detector = inject(InputModalityDetector);

  constructor() {
    this.detector.modalityDetected
      .pipe(takeUntilDestroyed())
      .subscribe((modality: InputModality) => {
        // 'keyboard' | 'mouse' | 'touch' | null
        document.body.classList.toggle('keyboard-user', modality === 'keyboard');
      });
  }

  // Check current modality synchronously
  get isKeyboard() { return this.detector.mostRecentModality === 'keyboard'; }
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
  private hcDetector = inject(HighContrastModeDetector);
  isHighContrast = this.hcDetector.getHighContrastMode() !== HighContrastMode.NONE;
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
  title = input.required<string>();
  closed = output<void>();

  private liveAnnouncer = inject(LiveAnnouncer);
  private previousFocus = document.activeElement as HTMLElement;
  private titleId = `dialog-title-${crypto.randomUUID()}`;

  constructor() {
    this.liveAnnouncer.announce(`Dialog opened: ${this.title()}`);
  }

  close() {
    this.liveAnnouncer.announce('Dialog closed');
    this.previousFocus?.focus();
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
  focusOrigin = signal<FocusOrigin>(null);
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
  private router = inject(Router);
  private focusMonitor = inject(FocusMonitor);
  private mainContent = viewChild.required<ElementRef>('mainContent');

  constructor() {
    this.router.events.pipe(
      filter(e => e instanceof NavigationEnd),
      takeUntilDestroyed(),
    ).subscribe(() => {
      this.focusMonitor.focusVia(this.mainContent(), 'program');
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
  private liveAnnouncer = inject(LiveAnnouncer);
  isLoading = signal(true);
  content = signal('');

  async load() {
    this.isLoading.set(true);
    this.liveAnnouncer.announce('Loading content');
    // ... fetch data
    this.isLoading.set(false);
    this.liveAnnouncer.announce('Content loaded');
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
onSort(column: string, direction: string) {
  this.tableAnnouncement.set(`Table sorted by ${column}, ${direction}`);
}

onFilter(count: number) {
  this.tableAnnouncement.set(`Showing ${count} results`);
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
