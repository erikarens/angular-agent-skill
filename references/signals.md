# Angular Signals — Detailed Reference

## Table of Contents

- [WritableSignal API](#writablesignal-api)
- [Computed Signals](#computed-signals)
- [Signal Inputs](#signal-inputs)
- [Signal Outputs](#signal-outputs)
- [Model Inputs](#model-inputs)
- [Signal Queries](#signal-queries)
- [linkedSignal](#linkedsignal)
- [resource / httpResource](#resource--httpresource)
- [effect and afterRenderEffect](#effect-and-afterrendereffect)
- [RxJS Interop](#rxjs-interop)
- [Migration from Decorators](#migration-from-decorators)

## WritableSignal API

```typescript
import { signal, computed, isSignal, isWritableSignal, untracked } from '@angular/core';

// Creation
const count = signal(0);
const user = signal<User | null>(null);

// Custom equality (default is Object.is)
const data = signal({ name: 'A' }, {
  equal: (a, b) => a.name === b.name,
});

// Read
count();          // 0

// Write
count.set(5);
count.update(v => v + 1);

// Read-only wrapper
const readOnly = count.asReadonly(); // Signal<number>, no set/update

// Type guards
isSignal(count);          // true
isWritableSignal(count);  // true
isWritableSignal(readOnly); // false
```

**Deep mutation caveat**: `asReadonly()` prevents `.set()`/`.update()` but does not prevent deep mutation of object/array values. Use immutable update patterns.

## Computed Signals

```typescript
const firstName = signal('John');
const lastName = signal('Doe');
const fullName = computed(() => `${firstName()} ${lastName()}`);
```

- Lazily evaluated — derivation runs only on first read
- Memoized — cached until a dependency changes
- Read-only — no `.set()` or `.update()`
- Dynamic dependencies — only signals actually read are tracked
- Use for all derived state

### Untracked reads

Prevent a signal from becoming a dependency:

```typescript
effect(() => {
  console.log(`User: ${currentUser()} count: ${untracked(counter)}`);
  // Only re-runs when currentUser changes, not counter
});
```

## Signal Inputs

Replace `@Input()` decorator with `input()` function:

```typescript
import { input, booleanAttribute, numberAttribute } from '@angular/core';

// Optional with default
value = input(0);                         // InputSignal<number>

// Optional without default
name = input<string>();                   // InputSignal<string | undefined>

// Required
label = input.required<string>();         // InputSignal<string>

// With transform
disabled = input(false, { transform: booleanAttribute });
count = input(0, { transform: numberAttribute });
trimmed = input('', { transform: (v: string) => v.trim() });

// With alias
value = input(0, { alias: 'sliderValue' });
// Template: <my-comp [sliderValue]="50" />
```

- `input()` returns `InputSignal<T>`, read by calling as function: `this.value()`
- Works natively in `computed()`, `effect()`, and templates
- Transform functions must be pure and statically analyzable
- `booleanAttribute` treats presence as true, `"false"` as false
- `numberAttribute` parses to number, NaN on failure

## Signal Outputs

Replace `@Output()` + `EventEmitter` with `output()`:

```typescript
import { output } from '@angular/core';

// Void event
closed = output<void>();
this.closed.emit();

// Typed event
valueChanged = output<number>();
this.valueChanged.emit(42);

// With alias
changed = output({ alias: 'valueChanged' });
```

- Returns `OutputEmitterRef<T>` with `.emit()` method
- Custom events do not bubble up the DOM
- Names are case-sensitive

### Programmatic subscription (dynamic components)

```typescript
const ref = viewContainerRef.createComponent(MyComponent);
ref.instance.someEvent.subscribe(data => console.log(data));
// Returns OutputRefSubscription with .unsubscribe()
```

## Model Inputs

Two-way bindable signals. Use for custom form controls or components that need to write back:

```typescript
import { model } from '@angular/core';

// Optional
checked = model(false);             // ModelSignal<boolean>

// Required
value = model.required<number>();   // ModelSignal<number>

// Write back
this.checked.set(true);
this.checked.update(v => !v);
```

Parent usage:

```html
<my-checkbox [(checked)]="isEnabled" />
```

- Automatically creates a `checkedChange` output event
- Do not support transforms (unlike `input()`)
- Can be aliased: `model(false, { alias: 'isChecked' })`
- Use `input()` when component should not write back; use `model()` when it should

## Signal Queries

Replace `@ViewChild`, `@ViewChildren`, `@ContentChild`, `@ContentChildren`:

```typescript
import { viewChild, viewChildren, contentChild, contentChildren } from '@angular/core';

// Optional view query
header = viewChild(HeaderComponent);           // Signal<HeaderComponent | undefined>
headerEl = viewChild('myRef');                 // By template ref variable
headerTpl = viewChild('myRef', { read: TemplateRef }); // Read as TemplateRef

// Required view query
header = viewChild.required(HeaderComponent);  // Signal<HeaderComponent>

// Multiple view children
items = viewChildren(ItemComponent);           // Signal<readonly ItemComponent[]>

// Content queries (same pattern)
toggle = contentChild(ToggleComponent);
menuItems = contentChildren(MenuItemComponent);
deepItems = contentChildren(MenuItemComponent, { descendants: true });
```

- Return signals — automatically reactive in `computed()`, `effect()`, templates
- No need for `ngAfterViewInit` / `ngAfterContentInit` lifecycle hooks
- `contentChildren` defaults to direct children only; use `{ descendants: true }` for all

## linkedSignal

Writable signal that resets when source signals change. Use when state depends on other state but can also be manually overridden:

```typescript
import { linkedSignal } from '@angular/core';

// Shorthand: resets whenever options() changes
selectedOption = linkedSignal(() => this.options()[0]);

// Full form: preserve previous selection
selectedOption = linkedSignal({
  source: this.options,
  computation: (newOptions, previous) => {
    // previous?.value = previous linkedSignal value
    // previous?.source = previous source value
    return newOptions.find(o => o.id === previous?.value.id) ?? newOptions[0];
  },
});

// Custom equality
copy = linkedSignal(() => this.original(), {
  equal: (a, b) => a.id === b.id,
});

// Still writable
selectedOption.set(someOtherOption);
selectedOption.update(v => /* ... */);
```

**When to choose**:
- `computed()` — derived, read-only state
- `linkedSignal()` — derived state that the user can also manually override
- `signal()` — independent, writable state

## resource / httpResource

Experimental APIs for async data in signal-based code.

### resource

```typescript
import { resource } from '@angular/core';

userResource = resource({
  params: () => ({ id: this.userId() }),  // Reactive params (like computed)
  loader: async ({ params, abortSignal }) => {
    const res = await fetch(`/api/users/${params.id}`, { signal: abortSignal });
    return res.json();
  },
});
```

**Reading state**:

| Property | Type | Description |
|----------|------|-------------|
| `.value()` | `T \| undefined` | Latest resolved value |
| `.hasValue()` | `boolean` | Type guard, narrows away `undefined` |
| `.status()` | `ResourceStatus` | `'idle'`, `'loading'`, `'reloading'`, `'resolved'`, `'error'`, `'local'` |
| `.isLoading()` | `boolean` | True during loading/reloading |
| `.error()` | `unknown` | Latest error or undefined |

**Actions**: `.reload()`, `.set(value)`, `.update(fn)`

Returning `undefined` from `params` prevents loader execution.

### rxResource (RxJS variant)

```typescript
import { rxResource } from '@angular/core/rxjs-interop';

userResource = rxResource({
  params: () => ({ id: this.userId() }),
  stream: ({ params }) => this.http.get<User>(`/api/users/${params.id}`),
});
```

Same API as `resource`, but `stream` returns an Observable instead of a Promise.

## effect and afterRenderEffect

### effect

Runs side effects when tracked signals change. Requires injection context:

```typescript
constructor() {
  // Basic
  effect(() => {
    console.log('Count:', this.count());
  });

  // With cleanup
  effect((onCleanup) => {
    const sub = someObservable.subscribe();
    onCleanup(() => sub.unsubscribe());
  });

  // Outside constructor — pass injector
  // this.injector = inject(Injector);
  // effect(() => { ... }, { injector: this.injector });
}

// Manual destroy
const ref = effect(() => { /* ... */ });
ref.destroy();
```

**Rules**:
- Never propagate state in effects (no `signal.set()` inside effect)
- Use for: logging, localStorage sync, analytics, DOM interop
- Use `computed()` or `linkedSignal()` for derived state instead

### afterRenderEffect

For DOM operations after Angular renders:

```typescript
constructor() {
  afterRenderEffect(() => {
    this.chart.updateData(this.chartData());
  });

  // Phased (prevents layout thrashing)
  afterRenderEffect({
    earlyRead: () => this.elementHeight = this.el.nativeElement.offsetHeight,
    write: () => this.renderer.setStyle(this.el.nativeElement, 'height', '100px'),
  });
}
```

Phases execute in order: `earlyRead` → `write` → `mixedReadWrite` → `read`. Only runs in the browser, not during SSR.

### afterNextRender

Runs once after the next render cycle. Use for one-time browser initialization:

```typescript
constructor() {
  afterNextRender(() => {
    this.chart = initializeChart(this.canvas().nativeElement);
  });
}
```

## RxJS Interop

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';
```

### toSignal — Observable to Signal

```typescript
// With initial value
counter = toSignal(this.counter$, { initialValue: 0 });  // Signal<number>

// Without initial value
counter = toSignal(this.counter$);  // Signal<number | undefined>

// BehaviorSubject (synchronous)
counter = toSignal(this.counter$, { requireSync: true }); // Signal<number>

// Custom equality
temp = toSignal(this.temp$, {
  equal: (a, b) => a.celsius === b.celsius,
});

// Manual cleanup (for self-completing observables)
data = toSignal(this.data$, { manualCleanup: true });
```

- Requires injection context
- Auto-unsubscribes on component destroy
- Errors thrown when reading signal if observable errors
- Reuse the returned signal; don't call `toSignal` repeatedly for the same observable

### toObservable — Signal to Observable

```typescript
query$ = toObservable(this.query);
results$ = this.query$.pipe(
  switchMap(q => this.http.get(`/search?q=${q}`))
);
```

- Uses internal effect + ReplaySubject
- Multiple rapid signal updates emit only the final stabilized value
- Requires injection context

## Migration from Decorators

| Old (Decorator) | New (Function) |
|-----------------|----------------|
| `@Input() value = 0` | `value = input(0)` |
| `@Input({ required: true }) name!: string` | `name = input.required<string>()` |
| `@Output() clicked = new EventEmitter<void>()` | `clicked = output<void>()` |
| `@ViewChild(Comp) comp!: Comp` | `comp = viewChild.required(Comp)` |
| `@ViewChildren(Comp) comps!: QueryList<Comp>` | `comps = viewChildren(Comp)` |
| `@ContentChild(Comp) child!: Comp` | `child = contentChild.required(Comp)` |

Key differences:
- Function-based APIs return signals, enabling reactive composition
- No need for lifecycle hooks to access query results
- `QueryList` replaced by `Signal<readonly T[]>` (no `.changes` observable needed)
- All function-based APIs must be called in class field initializers
