# Angular Development Skill

A comprehensive [Claude Code Agent Skill](https://docs.anthropic.com/en/docs/claude-code/skills) for modern Angular development. Provides expert guidance on signals, standalone components, built-in control flow, and TypeScript best practices.

## What It Does

This skill transforms Claude into an Angular expert that follows modern conventions and APIs. When activated, Claude will:

- Write components using **signal-based inputs, outputs, queries, and model inputs** instead of legacy decorators
- Use **built-in control flow** (`@if`, `@for`, `@switch`) instead of structural directives
- Apply **`ChangeDetectionStrategy.OnPush`** and signal-based state management
- Follow your project's folder structure conventions
- Use `inject()` for dependency injection, functional guards/interceptors, and modern routing patterns
- Optimize performance with `@defer`, `NgOptimizedImage`, and `takeUntilDestroyed()`

## Coverage

### Core Topics (SKILL.md)

| Area | What's Covered |
|------|---------------|
| **Signals** | `signal()`, `computed()`, `effect()`, `linkedSignal()`, `resource()` |
| **Component I/O** | `input()`, `input.required()`, `output()`, `model()` |
| **Queries** | `viewChild()`, `viewChildren()`, `contentChild()`, `contentChildren()` |
| **Templates** | `@if`/`@for`/`@switch`, `@defer` with triggers & prefetching |
| **DI & HTTP** | `inject()`, `provideHttpClient()`, functional interceptors, `firstValueFrom` |
| **Routing** | Lazy loading, functional guards, `withComponentInputBinding()`, `withViewTransitions()` |
| **SSR** | `RenderMode`, hydration, `afterNextRender()`, `withEventReplay()` |
| **App Config** | Full `app.config.ts` pattern with all modern providers |

### Reference Files (loaded on demand)

| File | Contents |
|------|----------|
| `references/signals.md` | Full signal API reference, `linkedSignal` patterns, `resource`/`rxResource`, `effect` cleanup, RxJS interop (`toSignal`/`toObservable`), decorator migration table |
| `references/templates.md` | Control flow details, `@defer` triggers & sub-blocks, `NgOptimizedImage`, SSR behavior, accessibility patterns, testing deferred views |

## Install

```bash
claude skill install /path/to/angular-development.skill
```

Or install directly from the folder:

```bash
claude skill install /path/to/angular-development/
```

## Example Interactions

**Creating a component:**
> "Create a user card component that displays a user's name and avatar"

Claude will generate a standalone, OnPush component with `input.required<User>()`, signal-based queries, and Tailwind styling.

**Data fetching:**
> "Add a service that fetches products from the API"

Claude will use `inject(HttpClient)`, `firstValueFrom`, and optionally `resource()` for signal-based async data.

**Lazy loading:**
> "Add a settings feature module with lazy-loaded routes"

Claude will create a feature folder with `loadComponent`, functional guards, and `@defer` blocks for heavy sub-components.

**State management:**
> "I need a filter dropdown that resets when the data source changes"

Claude will use `linkedSignal()` to create a writable signal that auto-resets when the source changes, preserving the selection when possible.

## Skill Structure

```
angular-development/
├── SKILL.md                  # Core instructions (~330 lines)
└── references/
    ├── signals.md             # Signal API deep-dive (~250 lines)
    └── templates.md           # Template patterns deep-dive (~200 lines)
```

## Angular Version

Built for **Angular 19+** with coverage of:
- Stable APIs: signals, standalone components, built-in control flow, `@defer`, signal inputs/outputs/queries/model
- Experimental APIs: `resource()`, `rxResource()`, `linkedSignal()`, zoneless change detection

## License

MIT
