---
name: angular-development
description: Expert Angular, TypeScript, and Tailwind development with modern signal-based patterns, standalone components, and built-in control flow. Use when working on Angular projects, creating components, services, directives, pipes, writing templates, managing state with signals, handling forms, routing, HTTP requests, testing, or any Angular-related code. Also use when reviewing Angular code, debugging Angular issues, or making architectural decisions in Angular applications.
---

# Angular Development

Expert guidance for building scalable Angular applications using modern APIs: signals, standalone components, built-in control flow, and strict TypeScript.

## Core Architecture

### Project Structure

```
src/
└── app/
    ├── core/           # Non-business features (layout, auth, guards)
    ├── features/       # Business domain features
    ├── shared/         # Reusable "dumb" components, pipes, directives
    ├── services/       # Global and feature-specific services
    ├── interfaces/     # TypeScript interfaces
    ├── types/          # Type definitions, DTOs (requests/, responses/)
    ├── enums/          # Enums
    ├── utils/          # Pure utility functions
    ├── app.component.ts
    ├── app.config.ts
    └── app.routes.ts
```

Feature folders follow: `pages/` (routed), `components/` (non-routed), `guards/`, `feature.routes.ts`.

### Coding Standards

- Single quotes, 2-space indent, `const` by default
- Kebab-case file names with Angular suffixes (`.component.ts`, `.service.ts`)
- Strict typing, never use `any` — define explicit types
- Optional chaining (`?.`) and nullish coalescing (`??`)
- Template literals for string interpolation
- File structure: imports, class, properties, lifecycle, public methods, private methods

## Signals (State Management)

Use Angular's signal system as the primary state management approach.

### Signal Primitives

```typescript
// Writable signal
count = signal(0);
count.set(3);
count.update(v => v + 1);

// Read-only exposure
private readonly _count = signal(0);
readonly count = this._count.asReadonly();

// Computed (lazy, memoized, read-only)
doubleCount = computed(() => this.count() * 2);

// Custom equality for objects
data = signal(['test'], { equal: (a, b) => a.length === b.length });
```

### Signal Inputs, Outputs & Queries

Use function-based APIs instead of decorators:

```typescript
// Inputs
value = input(0);                                    // InputSignal<number>
name = input<string>();                              // InputSignal<string | undefined>
label = input.required<string>();                    // Required, no undefined
disabled = input(false, { transform: booleanAttribute });

// Outputs
closed = output<void>();                             // OutputEmitterRef<void>
valueChanged = output<number>();
this.valueChanged.emit(42);

// Model inputs (two-way binding)
checked = model(false);                              // ModelSignal<boolean>
// Parent: <my-comp [(checked)]="isChecked" />

// Queries
header = viewChild(HeaderComponent);                 // Signal<HeaderComponent | undefined>
items = viewChildren(ItemComponent);                 // Signal<readonly ItemComponent[]>
toggle = contentChild.required(ToggleComponent);     // Signal<ToggleComponent>
```

### linkedSignal

Writable signal that resets when dependencies change. Use for selections that depend on changing data:

```typescript
// Shorthand
selectedOption = linkedSignal(() => this.options()[0]);

// Full form — preserves selection across data changes
selectedOption = linkedSignal({
  source: this.options,
  computation: (newOptions, previous) =>
    newOptions.find(o => o.id === previous?.value.id) ?? newOptions[0],
});
```

### resource / httpResource

Integrate async data into signals. Experimental but ready for use:

```typescript
userResource = resource({
  params: () => ({ id: this.userId() }),
  loader: ({ params, abortSignal }) =>
    fetch(`/api/users/${params.id}`, { signal: abortSignal }).then(r => r.json()),
});

// Access: userResource.value(), .status(), .isLoading(), .error(), .hasValue()
// Actions: userResource.reload(), .set(), .update()
```

### effect

For side effects only — logging, localStorage sync, DOM interop. Never use for state propagation (use `computed` or `linkedSignal` instead):

```typescript
constructor() {
  effect((onCleanup) => {
    const user = this.currentUser();
    const timer = setTimeout(() => console.log(user), 1000);
    onCleanup(() => clearTimeout(timer));
  });
}
```

### RxJS Interop

```typescript
// Observable → Signal
counter = toSignal(this.counter$, { initialValue: 0 });
// Use requireSync: true for BehaviorSubject

// Signal → Observable
query$ = toObservable(this.query);
```

For detailed signal patterns, see [references/signals.md](references/signals.md).

## Templates

### Built-in Control Flow

Always use the built-in block syntax. Never use `*ngIf`, `*ngFor`, `*ngSwitch`:

```html
@if (user(); as user) {
  <h1>{{ user.name }}</h1>
} @else {
  <p>Loading...</p>
}

@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items found.</li>
}

@switch (role()) {
  @case ('admin') { <admin-panel /> }
  @case ('editor') { <editor-panel /> }
  @default { <viewer-panel /> }
}
```

`@for` requires `track`. Use a unique identifier; use `$index` only for static lists. Available context: `$index`, `$first`, `$last`, `$even`, `$odd`, `$count`.

### Deferrable Views

Reduce initial bundle size by deferring non-critical content:

```html
@defer (on viewport; prefetch on idle) {
  <heavy-chart [data]="chartData()" />
} @placeholder (minimum 200ms) {
  <div class="skeleton"></div>
} @loading (after 100ms; minimum 500ms) {
  <spinner />
} @error {
  <p>Failed to load chart.</p>
}
```

Triggers: `on idle`, `on viewport`, `on interaction`, `on hover`, `on timer(500ms)`, `on immediate`, `when condition`.

For detailed template patterns, see [references/templates.md](references/templates.md).

## Components & DI

### Standalone by Default

All components are standalone by default (Angular 19+). Import dependencies directly:

```typescript
@Component({
  selector: 'app-user-card',
  imports: [DatePipe, RouterLink],
  templateUrl: './user-card.component.html',
  styleUrl: './user-card.component.css',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent {
  user = input.required<User>();
  selected = output<void>();
}
```

### Dependency Injection

Use `inject()` function, not constructor injection:

```typescript
export class UserService {
  private readonly http = inject(HttpClient);
  private readonly config = inject(ConfigService);
}
```

Services use `providedIn: 'root'` for tree-shakeable singletons.

### HTTP

Convert observables to promises with `firstValueFrom`:

```typescript
async getUser(id: string): Promise<User> {
  return firstValueFrom(
    this.http.get<User>(`/api/users/${id}`)
  );
}
```

Configure with functional API:

```typescript
provideHttpClient(
  withInterceptors([authInterceptor]),
  withFetch(),  // Use Fetch API, required for SSR streaming
)
```

### Functional Interceptors

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (token) {
    req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
  }
  return next(req);
};
```

### Lifecycle & Cleanup

- Use `afterNextRender()` / `afterRenderEffect()` for DOM operations (browser-only)
- Prefer `DestroyRef` + `takeUntilDestroyed()` for subscription cleanup:

```typescript
// In constructor (injection context) — no argument needed
this.route.params.pipe(takeUntilDestroyed()).subscribe(/*...*/);

// Outside constructor — pass DestroyRef
private destroyRef = inject(DestroyRef);
ngOnInit() {
  this.obs$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe(/*...*/);
}
```

- Render phases: `earlyRead` → `write` → `mixedReadWrite` → `read`

## Routing

Use functional guards and resolvers. Lazy-load feature routes:

```typescript
export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/pages/dashboard.component'),
    canActivate: [() => inject(AuthService).isAuthenticated()],
  },
  {
    path: 'user/:id',
    loadComponent: () => import('./features/user/pages/user-detail.component'),
    resolve: { user: (route: ActivatedRouteSnapshot) => inject(UserService).getUser(route.paramMap.get('id')!) },
  },
];
```

Enable `withComponentInputBinding()` to automatically bind route params, query params, and resolve data to component inputs:

```typescript
// app.config.ts
provideRouter(routes, withComponentInputBinding(), withViewTransitions())

// Component — 'id' auto-bound from route param ':id', 'user' from resolve
export class UserDetailComponent {
  id = input<string>();
  user = input<User>();
}
```

## App Configuration

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding(), withViewTransitions()),
    provideHttpClient(withInterceptors([authInterceptor]), withFetch()),
    provideAnimationsAsync(),
    provideClientHydration(withEventReplay()),
    // provideExperimentalZonelessChangeDetection(), // Zoneless (experimental)
  ],
};
```

## Performance

- `ChangeDetectionStrategy.OnPush` on all components
- `track` with unique IDs in `@for` loops
- `@defer` for below-fold and heavy components
- `NgOptimizedImage` for all images with `priority` on LCP image
- Pure pipes for repeated template computations
- Signals automatically optimize re-renders with granular tracking
- `takeUntilDestroyed()` to prevent subscription memory leaks

## Security

- Never use `innerHTML` — rely on Angular's built-in sanitization
- Sanitize dynamic content with Angular's DomSanitizer when necessary

## Testing

- Arrange-Act-Assert pattern
- Test services, components, and utilities with high coverage

## SSR / Hybrid Rendering

Configure per-route rendering in `app.routes.server.ts`:

```typescript
export const serverRoutes: ServerRoute[] = [
  { path: '', renderMode: RenderMode.Client },
  { path: 'about', renderMode: RenderMode.Prerender },
  { path: 'profile', renderMode: RenderMode.Server },
];
```

Use `afterNextRender()` for browser-only code. `@defer` blocks render `@placeholder` on server.

## References

- **[references/signals.md](references/signals.md)** — Detailed signal API reference, patterns for resource, linkedSignal, effect cleanup, and RxJS interop
- **[references/templates.md](references/templates.md)** — Control flow details, @defer triggers, SSR considerations, and template best practices
