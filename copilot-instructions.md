# Copilot Instructions

> `AGENTS.md` does not exist yet. These instructions are the sole source of truth.

---

## 1. Operating mode

Work strictly in **Plan → Tasks → Code → Verification** sequence.

- **Plan**: Understand the full scope before touching any file.
- **Tasks**: Break the work into atomic, reviewable steps.
- **Code**: Implement only what was planned.
- **Verification**: Check that the output matches the request, runs, and passes tests.

**Never** refactor, migrate, rename, or change tooling unless explicitly instructed.  
**Never** modify files unrelated to the current task.

---

## 2. Tech stack — respect it strictly

| Layer | Technology |
|---|---|
| Framework | Angular 19 |
| Language | TypeScript (strict mode) |
| Components | Standalone only (no NgModules) |
| State management | NgRx Store + NgRx ComponentStore + Signals |
| Reactivity | RxJS Observables + Signals + `async` pipe — use whichever fits the context |
| HTTP | HttpClient + interceptors |
| Micro-frontend | Native Federation |
| Design system | Web Components (Lit Element) injected into Angular templates |
| Tests | Karma + Jasmine + ng-mocks — 100% coverage target |
| Linting | ESLint + Stylelint + Prettier |
| Package manager | npm |
| Commits | Conventional Commits |

Do **not** introduce new libraries or tools without explicit approval.

---

## 3. Angular architecture

- **Standalone components exclusively** — never create or reference NgModules.
- Use **`inject()`** for all dependency injection — never constructor injection.
- Routing: use both **`loadComponent()`** and **`loadChildren()`** depending on context (single component vs feature route group).
- Structure: one folder per feature, one file per concern (`component` / `service` / `store` / `spec`).
- Barrel files (`index.ts`) are used — always export new public symbols through the nearest `index.ts`.

### Native Federation
- Do **not** touch `federation.config.ts` or `federation.manifest.json` unless explicitly asked.
- Do not move or rename files exposed as federation remotes without explicit instruction.
- Always check if a symbol is exposed via federation before refactoring its path or name.

---

## 4. Lit Element / Web Components design system

This project consumes a **Lit Element web component design system** injected into Angular templates.

- Use web components via their **custom element tag names** (e.g. `<ds-button>`, `<ds-input>`).
- Add **`CUSTOM_ELEMENTS_SCHEMA`** to the component's `schemas` array when using web components in a standalone component.
- Never recreate or wrap a design system component in Angular — use the existing web component.
- Pass data via **attributes** (string) or **properties** (object/array — set via `[prop]="value"` binding).
- Listen to custom events via `(eventName)="handler($event)"`.
- Do not apply Angular animations or CDK overlays on top of design system components without explicit approval.
- Do not override design system CSS internals — use only the exposed CSS custom properties (design tokens).

---

## 5. State management

Use the right tool for the right scope:

| Scope | Tool |
|---|---|
| Global / shared app state | NgRx Store (actions / reducers / selectors / effects) |
| Feature / local component state | NgRx ComponentStore |
| Synchronous derived / computed state | Angular Signals (`computed`, `effect`) |

- Always define **actions**, **reducers**, **selectors**, and **effects** in separate files.
- Never access the store directly from a template — always go through a selector.
- Prefer `toSignal()` to bridge Observables and Signals when needed.

---

## 6. Reactivity

- Use **RxJS** for async operations, streams, and HTTP.
- Use **Signals** (`signal`, `computed`, `effect`) for synchronous local state and derived values.
- Use **`async` pipe** in templates to subscribe to Observables — avoid manual subscriptions in components.
- Always unsubscribe from manual subscriptions — prefer `takeUntilDestroyed()` over `ngOnDestroy`.

---

## 7. HTTP

- All HTTP calls go through **dedicated services** — never call `HttpClient` directly from a component.
- Use **interceptors** for: auth headers, error handling, loading state, logging.
- Type all HTTP responses explicitly — never use `any` for API payloads.
- Handle errors with `catchError` in effects or services, never silently.

---

## 8. TypeScript conventions

- **`any` is forbidden** unless there is absolutely no alternative — in that case, add a comment explaining why.
- Prefer `unknown` and narrow with type guards.
- Always type function parameters and return values explicitly.
- Prefer `interface` over `type` for object shapes.
- Use `readonly` on properties that must not be mutated.
- `tsconfig` strict mode must stay enabled at all times.

---

## 9. File and naming conventions

- Files: `kebab-case.type.ts` — e.g. `user.store.ts`, `auth.service.ts`, `header.component.ts`
- Classes: `PascalCase`
- Variables / functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Always export new public symbols through the nearest `index.ts` barrel file.
- No wildcard imports (`import * as`).

---

## 10. Tests — maximum coverage

Every **component**, **service**, **store**, **effect**, **selector**, and **pipe** must have a `.spec.ts`.

### Rules
- Use **ng-mocks** (`MockComponent`, `MockService`, `MockProvider`) for all dependencies.
- Mock web components with `MockComponent` or declare them via `CUSTOM_ELEMENTS_SCHEMA` in the test bed.
- Test structure: `describe` → `beforeEach` → `it` — one assertion per `it` when possible.
- Cover: happy path, edge cases, error states, boundary values.
- Do **not** modify existing tests unless the code they cover has changed.
- Do **not** skip or comment out failing tests — fix them or flag them explicitly.
- Target: **maximum code coverage** on branches, statements, functions, and lines.

---

## 11. Accessibility (RGAA 4.1 / WCAG 2.1 AA)

Every generated UI must:

- Use semantic HTML elements (`<nav>`, `<main>`, `<button>`, `<label>`, etc.).
- Provide `aria-label` or `aria-labelledby` on interactive elements without visible text.
- Ensure full **keyboard navigability** (tab order, focus management, skip links).
- Maintain color contrast ratio **≥ 4.5:1** for text (WCAG 1.4.3).
- Associate every form field with a `<label>` via `for`/`id` or `aria-labelledby`.
- Never use `role` as a substitute for correct semantic elements.
- Provide `alt` text for all non-decorative images.
- When using Lit Element web components, verify that the component exposes the necessary ARIA attributes — add them at the Angular level if not.

---

## 12. Code quality

- No `console.log` in committed code — use a logging service.
- No commented-out code blocks — delete dead code, use Git history to recover.
- No hardcoded user-facing strings — use i18n keys or constants.
- CSS/SCSS: follow BEM or the existing convention in the file being edited.
- All ESLint, Stylelint, and Prettier rules must pass — do not add disable comments without justification.

---

## 13. Git — Conventional Commits

Format: `<type>(<scope>): <short description>`

| Type | Usage |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change without behavior change |
| `test` | Adding or fixing tests |
| `chore` | Build, config, dependencies |
| `docs` | Documentation only |
| `style` | Formatting, linting (no logic change) |

- Scope = feature or module name (e.g. `auth`, `user`, `navigation`, `ds`).
- Description: imperative, lowercase, no period at the end.
- One logical change per commit.

Example: `feat(user): add profile update form with ds-input components`

---

## 14. What Copilot must never do

- Add or remove npm dependencies autonomously.
- Modify `angular.json`, `tsconfig*.json`, `eslint.config.*`, `stylelint.config.*`, or `prettier.config.*`.
- Modify `federation.config.ts` or `federation.manifest.json`.
- Use `any`, `@ts-ignore`, or `@ts-expect-error` without an explicit justification comment.
- Recreate or wrap a Lit Element design system component in Angular.
- Override design system CSS internals.
- Create files outside the scope of the current task.
- Remove or bypass existing test coverage.
