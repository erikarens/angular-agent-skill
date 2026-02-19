# Angular Templates — Detailed Reference

## Table of Contents

- [Built-in Control Flow](#built-in-control-flow)
- [Deferrable Views](#deferrable-views)
- [Image Optimization](#image-optimization)
- [Template Best Practices](#template-best-practices)

## Built-in Control Flow

Always use built-in control flow blocks. Never use `*ngIf`, `*ngFor`, `*ngSwitch` structural directives.

### @if / @else if / @else

```html
@if (user(); as user) {
  <h1>Welcome, {{ user.name }}</h1>
} @else if (isLoading()) {
  <app-spinner />
} @else {
  <p>Please log in</p>
}
```

The `as` keyword stores the expression result in a local variable — useful for deeply nested property access.

### @for

```html
@for (item of items(); track item.id; let i = $index, isLast = $last) {
  <div [class.border-b]="!isLast">
    {{ i + 1 }}. {{ item.name }}
  </div>
} @empty {
  <p>No items found.</p>
}
```

**`track` is required.** Rules:
- Use a unique identifier (`item.id`, `item.uuid`) when available
- Use `$index` only for static, non-reorderable lists
- Never use `track item` (object reference) — causes performance degradation

**Context variables**:

| Variable | Type | Description |
|----------|------|-------------|
| `$index` | `number` | Current index |
| `$count` | `number` | Total items |
| `$first` | `boolean` | Is first item |
| `$last` | `boolean` | Is last item |
| `$even` | `boolean` | Even index |
| `$odd` | `boolean` | Odd index |

**View reuse**: Unlike `*ngFor`, `@for` preserves views when the tracked property stays the same even if the object reference changes. This updates bindings without destroying/recreating DOM.

### @switch / @case / @default

```html
@switch (status()) {
  @case ('active') {
    <span class="text-green-500">Active</span>
  }
  @case ('pending') {
    <span class="text-yellow-500">Pending</span>
  }
  @default {
    <span class="text-gray-400">Unknown</span>
  }
}
```

- Uses strict equality (`===`)
- No fallthrough — no `break` needed
- Multiple consecutive `@case` blocks can share the same template:

```html
@case ('reviewer')
@case ('editor') {
  <app-editor-panel />
}
```

## Deferrable Views

Defer loading of non-critical UI to reduce initial bundle size. Only standalone components/directives/pipes can be deferred.

### Basic structure

```html
@defer (on viewport; prefetch on idle) {
  <heavy-component />
} @placeholder (minimum 200ms) {
  <div class="h-64 bg-gray-100 animate-pulse"></div>
} @loading (after 100ms; minimum 500ms) {
  <app-spinner />
} @error {
  <p>Failed to load component.</p>
}
```

### Sub-blocks

| Block | Purpose | Dependencies |
|-------|---------|-------------|
| `@placeholder` | Shown before trigger fires | Eagerly loaded |
| `@loading` | Shown while fetching JS | Eagerly loaded |
| `@error` | Shown if loading fails | Eagerly loaded |

- `@placeholder` accepts `minimum` time to prevent flickering
- `@loading` accepts `after` (delay before showing) and `minimum` (min display time)

### Triggers

| Trigger | Behavior |
|---------|----------|
| `on idle` | Default. Loads when browser is idle (`requestIdleCallback`) |
| `on viewport` | Loads when element enters viewport (IntersectionObserver) |
| `on interaction` | Loads on `click` or `keydown` |
| `on hover` | Loads on `mouseover` or `focusin` |
| `on immediate` | Loads right after non-deferred content renders |
| `on timer(500ms)` | Loads after specified delay |
| `when condition` | Loads when expression becomes truthy (one-time) |

### Viewport with options

```html
<!-- Watch a specific element -->
@defer (on viewport(targetElement)) {
  <heavy-cmp />
}
<div #targetElement>Scroll to load</div>

<!-- With IntersectionObserver options -->
@defer (on viewport({ trigger: targetElement, rootMargin: '100px', threshold: 0.5 })) {
  <heavy-cmp />
}
```

### Prefetching

Separate prefetch triggers let you download JS ahead of display:

```html
@defer (on interaction; prefetch on idle) {
  <!-- JS fetched on idle, component rendered on interaction -->
  <heavy-cmp />
}
```

### Combining triggers

Multiple triggers fire on first match:

```html
@defer (on viewport; on timer(5s)) {
  <!-- Loads when visible OR after 5 seconds, whichever comes first -->
  <heavy-cmp />
}
```

### SSR behavior

- `@defer` renders only `@placeholder` during SSR
- Triggers are disabled on the server
- Use HMR `--no-hmr` flag to test trigger behavior in dev (HMR eagerly loads all deferred chunks)

### Accessibility

Wrap `@defer` in ARIA live regions for screen reader announcements:

```html
<div aria-live="polite" aria-atomic="true">
  @defer (on viewport) {
    <user-profile />
  } @placeholder {
    <p>Loading profile...</p>
  }
</div>
```

### Testing

```typescript
TestBed.configureTestingModule({
  deferBlockBehavior: DeferBlockBehavior.Manual,
});

const fixture = TestBed.createComponent(MyComponent);
const deferBlock = (await fixture.getDeferBlocks())[0];
await deferBlock.render(DeferBlockState.Loading);
// assert loading state
await deferBlock.render(DeferBlockState.Complete);
// assert complete state
```

## Image Optimization

Always use `NgOptimizedImage` for images:

```html
<img ngSrc="hero.jpg" width="800" height="400" priority />
<img ngSrc="thumbnail.jpg" width="200" height="200" loading="lazy" />
```

- `priority` for above-the-fold LCP images (disables lazy loading, adds `fetchpriority="high"`)
- Automatic `srcset` generation with configured image loader
- Warns about missing `width`/`height` to prevent CLS

## Template Best Practices

1. Use `async` pipe for observables in templates OR convert to signals with `toSignal()`
2. Use pure pipes for repeated template computations
3. Never call functions in templates (use computed signals or pipes instead)
4. Use `@defer` for below-the-fold or heavy components
5. Nest `@defer` blocks with different triggers to prevent cascading loads
6. Avoid `@defer` for content visible in the initial viewport (causes layout shift)
7. Use semantic HTML elements and ARIA attributes for accessibility
