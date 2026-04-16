---
name: angular
description: Angular-specific additions for scaffold, test, debug, and document skills — use alongside base skills when working in an Angular project
license: MIT
compatibility: opencode
metadata:
  tags: angular, frontend, typescript
---

## What I do

Extend the base skills (`scaffold`, `test`, `debug`, `document`) with Angular-specific knowledge:
Angular CLI, component patterns, NgModule vs standalone, RxJS, TestBed, and Angular DevTools.

---

## Scaffold — Angular additions

### Use the Angular CLI, not manual file creation

```bash
# Component (standalone, Angular 14+)
ng generate component features/user-profile --standalone

# Component (NgModule-based)
ng generate component features/user-profile

# Service
ng generate service services/user

# Route guard
ng generate guard guards/auth --implements CanActivate

# HTTP interceptor (Angular 15+)
ng generate interceptor interceptors/auth

# Pipe
ng generate pipe pipes/format-date

# Module (if using NgModule pattern)
ng generate module features/user --routing
```

### Standalone vs NgModule — how to determine which to use

```typescript
// Check angular.json or existing components for the pattern in use
// Standalone (Angular 14+) — no NgModule required
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, RouterModule],  // import directly in component
  templateUrl: './user-card.component.html',
})
export class UserCardComponent { }

// NgModule-based — must be declared in a module
@Component({ selector: 'app-user-card', templateUrl: '...' })
export class UserCardComponent { }
// AND added to the module:
@NgModule({ declarations: [UserCardComponent], exports: [UserCardComponent] })
export class UserModule { }
```

### Wiring checklist per artifact type

| Artifact | Standalone wiring | NgModule wiring |
|---|---|---|
| Component | Import in parent component's `imports: []` | Declare in module `declarations: []` |
| Service | `providedIn: 'root'` or `providers: []` in `app.config.ts` | `providers: []` in module |
| Route guard | Register in route definition | Register in route definition |
| Interceptor | `provideHttpClient(withInterceptors([authInterceptor]))` in `app.config.ts` | `HTTP_INTERCEPTORS` provider in module |

---

## Test — Angular additions

### Setup pattern

```typescript
import { ComponentFixture, TestBed, fakeAsync, tick } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { RouterTestingModule } from '@angular/router/testing';

describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [
        UserCardComponent,          // standalone component imports itself
        HttpClientTestingModule,    // mock HTTP
        RouterTestingModule,        // mock router
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();       // trigger ngOnInit
  });

  it('should display the user name', () => {
    component.user = { id: '1', name: 'Anna' };
    fixture.detectChanges();       // re-render after input change
    const el = fixture.nativeElement.querySelector('[data-testid="user-name"]');
    expect(el.textContent).toBe('Anna');
  });
});
```

### Testing async / RxJS

```typescript
// fakeAsync + tick for Observable-based async
it('loads users on init', fakeAsync(() => {
  const userService = TestBed.inject(UserService);
  spyOn(userService, 'getUsers').and.returnValue(of([{ id: '1', name: 'Anna' }]));

  fixture.detectChanges();  // trigger ngOnInit
  tick();                   // flush async
  fixture.detectChanges();  // re-render

  expect(component.users.length).toBe(1);
}));

// Testing HTTP with HttpTestingController
it('fetches users from /api/users', () => {
  const httpMock = TestBed.inject(HttpTestingController);
  const service = TestBed.inject(UserService);

  service.getUsers().subscribe(users => {
    expect(users.length).toBe(1);
  });

  const req = httpMock.expectOne('/api/users');
  expect(req.request.method).toBe('GET');
  req.flush([{ id: '1', name: 'Anna' }]);
  httpMock.verify();  // assert no unexpected requests
});
```

### What to mock in Angular tests

- **Always mock**: `HttpClient` (use `HttpClientTestingModule`), `Router` (use `RouterTestingModule`), external services
- **Never mock**: Angular internals (`ChangeDetectorRef`, `ElementRef`), the component under test, `CommonModule`
- **Prefer spies over full mocks** for services: `spyOn(service, 'method').and.returnValue(of(data))`

---

## Debug — Angular additions

### Angular DevTools (browser extension)

1. Install Angular DevTools (Chrome/Firefox extension)
2. Open DevTools → **Angular** tab
3. Use **Component tree** to inspect `@Input` values and component state
4. Use **Profiler** to record change detection cycles — find components that re-render unexpectedly

### Debugging change detection

```typescript
// Component not updating? Force change detection manually to test:
import { ChangeDetectorRef } from '@angular/core';
constructor(private cdr: ChangeDetectorRef) {}
this.cdr.detectChanges();  // if this fixes it, you have a change detection zone issue

// Using OnPush? The component only updates when:
// - An @Input reference changes (not mutation)
// - An async pipe emits
// - You call cdr.markForCheck()
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })
```

### Debugging RxJS

```typescript
// Add tap() to inspect values mid-stream without changing behaviour
import { tap } from 'rxjs/operators';

this.userService.getUser(id).pipe(
  tap(user => console.log('before map:', user)),
  map(user => user.name),
  tap(name => console.log('after map:', name)),
).subscribe();

// Memory leak check — always unsubscribe
// Preferred: use async pipe in template (auto-unsubscribes)
// Or: takeUntilDestroyed() (Angular 16+)
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
this.stream$.pipe(takeUntilDestroyed()).subscribe(...);
```

### Common Angular error messages

| Error | Likely cause |
|---|---|
| `NullInjectorError: No provider for X` | Service not provided — add to `providers[]` or `providedIn: 'root'` |
| `Can't bind to 'ngModel'` | `FormsModule` not imported in the component/module |
| `NG0100: ExpressionChangedAfterItHasBeenChecked` | Property changed in `ngAfterViewInit` without `cdr.detectChanges()` |
| `NG0200: Circular dependency` | Two services inject each other — refactor into a third service |
| `ERROR TypeError: Cannot read properties of undefined` | Template renders before data loads — use `@if (data)` or `async` pipe |

---

## Document — Angular additions

### Component JSDoc conventions

```typescript
/**
 * Displays a user profile card.
 *
 * @example
 * <app-user-card [user]="currentUser" (editClicked)="onEdit($event)" />
 */
@Component({ selector: 'app-user-card', ... })
export class UserCardComponent {
  /** The user to display. Required. */
  @Input({ required: true }) user!: User;

  /** Emits the user ID when the edit button is clicked. */
  @Output() editClicked = new EventEmitter<string>();
}
```

### Service JSDoc conventions

```typescript
/** Manages user data and communicates with the /api/users endpoint. */
@Injectable({ providedIn: 'root' })
export class UserService {
  /**
   * Retrieves a user by ID.
   * @returns Observable that emits the user, or throws HttpErrorResponse if not found.
   */
  getUser(id: string): Observable<User> { ... }
}
```

---

## Signals (Angular 16+)

### Signal basics

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({ ... })
export class UserCardComponent {
  // Writable signal
  count = signal(0);

  // Computed signal (derived, read-only)
  doubled = computed(() => this.count() * 2);

  increment() {
    this.count.update(n => n + 1);   // update with function
    // or:
    this.count.set(5);               // set directly
  }
}

// effect — runs whenever a signal it reads changes
effect(() => {
  console.log('count changed:', this.count());
});
```

### Signal inputs and outputs (Angular 17+)

```typescript
import { input, output } from '@angular/core';

@Component({ selector: 'app-user-card', ... })
export class UserCardComponent {
  // Signal input (replaces @Input)
  user = input.required<User>();           // required
  showAvatar = input(true);                // optional with default

  // Signal output (replaces @Output + EventEmitter)
  editClicked = output<string>();

  onEdit() {
    this.editClicked.emit(this.user().id);
  }
}
```

### Migrating from @Input/@Output

| Old pattern | Signal equivalent |
|---|---|
| `@Input() user!: User` | `user = input.required<User>()` |
| `@Input() label = 'default'` | `label = input('default')` |
| `@Output() clicked = new EventEmitter<string>()` | `clicked = output<string>()` |

---

## New control flow syntax (Angular 17+)

Prefer the new built-in control flow over `*ngIf` / `*ngFor` directives:

```html
<!-- @if / @else — replaces *ngIf -->
@if (user()) {
  <app-user-card [user]="user()!" />
} @else {
  <p>Loading...</p>
}

<!-- @for — replaces *ngFor. track is required. -->
@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items.</li>
}

<!-- @switch — replaces [ngSwitch] -->
@switch (status()) {
  @case ('active') { <span class="green">Active</span> }
  @case ('inactive') { <span class="gray">Inactive</span> }
  @default { <span>Unknown</span> }
}
```

---

## inject() function

Use `inject()` instead of constructor injection for leaner code, especially in standalone components and functional guards:

```typescript
import { inject } from '@angular/core';

// In a component
@Component({ ... })
export class UserPageComponent {
  private userService = inject(UserService);
  private route = inject(ActivatedRoute);

  userId = toSignal(this.route.paramMap.pipe(map(p => p.get('id'))));
}

// In a functional route guard (replaces class-based CanActivate)
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true : router.createUrlTree(['/login']);
};
```

---

## Lazy loading routes

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.routes').then(m => m.USER_ROUTES),
  },
  {
    path: 'admin',
    loadComponent: () => import('./features/admin/admin.component').then(m => m.AdminComponent),
  },
];
```

---

## Build & run

```bash
ng serve                         # dev server at http://localhost:4200
ng serve --configuration=staging # use staging environment

ng build                         # production build to dist/
ng build --configuration=production

ng test                          # run unit tests with Karma
ng test --watch=false            # single run (CI)
ng e2e                           # end-to-end tests (Cypress or Playwright via @angular/cli)

ng lint                          # ESLint
```
