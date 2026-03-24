# Ionic Angular Testing

Unit testing and end-to-end testing patterns for Ionic Angular applications.

## Unit Testing Setup

Ionic Angular projects generated with the Ionic CLI include Jasmine and Karma for unit testing. Test files follow the convention `*.spec.ts` and live alongside source files.

Run tests:

```bash
ng test
```

## Testing Pages (Components)

### Standalone Page

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { HomePage } from './home.page';

describe('HomePage', () => {
  let component: HomePage;
  let fixture: ComponentFixture<HomePage>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [HomePage],
    }).compileComponents();

    fixture = TestBed.createComponent(HomePage);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### NgModule Page

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { IonicModule } from '@ionic/angular';
import { HomePage } from './home.page';

describe('HomePage', () => {
  let component: HomePage;
  let fixture: ComponentFixture<HomePage>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [HomePage],
      imports: [IonicModule.forRoot()],
    }).compileComponents();

    fixture = TestBed.createComponent(HomePage);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## Testing Services

### Basic Service

```typescript
import { TestBed } from '@angular/core/testing';
import { DataService } from './data.service';

describe('DataService', () => {
  let service: DataService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(DataService);
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should add an item', () => {
    service.add({ id: 1, name: 'Test' });
    expect(service.getAll().length).toBe(1);
  });
});
```

### Service with HTTP

```typescript
import { TestBed } from '@angular/core/testing';
import { provideHttpClient } from '@angular/common/http';
import { provideHttpClientTesting, HttpTestingController } from '@angular/common/http/testing';
import { ItemService, Item } from './item.service';

describe('ItemService', () => {
  let service: ItemService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [provideHttpClient(), provideHttpClientTesting()],
    });
    service = TestBed.inject(ItemService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch all items', () => {
    const mockItems: Item[] = [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
    ];

    service.getAll().subscribe((items) => {
      expect(items.length).toBe(2);
      expect(items).toEqual(mockItems);
    });

    const req = httpMock.expectOne('https://api.example.com/items');
    expect(req.request.method).toBe('GET');
    req.flush(mockItems);
  });
});
```

## Testing Components with Dependencies

Mock services using `jasmine.createSpyObj`:

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ItemListPage } from './item-list.page';
import { ItemService } from '../services/item.service';
import { of } from 'rxjs';

describe('ItemListPage', () => {
  let component: ItemListPage;
  let fixture: ComponentFixture<ItemListPage>;
  let itemServiceSpy: jasmine.SpyObj<ItemService>;

  beforeEach(async () => {
    itemServiceSpy = jasmine.createSpyObj('ItemService', ['getAll', 'delete']);
    itemServiceSpy.getAll.and.returnValue(of([{ id: 1, name: 'Test Item' }]));

    await TestBed.configureTestingModule({
      imports: [ItemListPage],
      providers: [{ provide: ItemService, useValue: itemServiceSpy }],
    }).compileComponents();

    fixture = TestBed.createComponent(ItemListPage);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should load items on enter', () => {
    component.ionViewWillEnter();
    expect(itemServiceSpy.getAll).toHaveBeenCalled();
  });
});
```

## Testing Navigation

Mock `NavController` for navigation tests:

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { NavController } from '@ionic/angular';
import { HomePage } from './home.page';

describe('HomePage', () => {
  let component: HomePage;
  let fixture: ComponentFixture<HomePage>;
  let navCtrlSpy: jasmine.SpyObj<NavController>;

  beforeEach(async () => {
    navCtrlSpy = jasmine.createSpyObj('NavController', ['navigateForward', 'back']);

    await TestBed.configureTestingModule({
      imports: [HomePage],
      providers: [{ provide: NavController, useValue: navCtrlSpy }],
    }).compileComponents();

    fixture = TestBed.createComponent(HomePage);
    component = fixture.componentInstance;
  });

  it('should navigate to detail page', () => {
    component.goToDetail(42);
    expect(navCtrlSpy.navigateForward).toHaveBeenCalledWith('/detail/42');
  });
});
```

## Testing Pipes

```typescript
import { CurrencyFormatPipe } from './currency-format.pipe';

describe('CurrencyFormatPipe', () => {
  const pipe = new CurrencyFormatPipe();

  it('should format a number as currency', () => {
    expect(pipe.transform(1234.5)).toBe('$1,234.50');
  });
});
```

## Testing Guards

```typescript
import { TestBed } from '@angular/core/testing';
import { Router } from '@angular/router';
import { authGuard } from './auth.guard';
import { AuthService } from '../services/auth.service';

describe('authGuard', () => {
  let authServiceSpy: jasmine.SpyObj<AuthService>;
  let routerSpy: jasmine.SpyObj<Router>;

  beforeEach(() => {
    authServiceSpy = jasmine.createSpyObj('AuthService', ['isAuthenticated']);
    routerSpy = jasmine.createSpyObj('Router', ['createUrlTree']);

    TestBed.configureTestingModule({
      providers: [
        { provide: AuthService, useValue: authServiceSpy },
        { provide: Router, useValue: routerSpy },
      ],
    });
  });

  it('should allow access when authenticated', () => {
    authServiceSpy.isAuthenticated.and.returnValue(true);
    const result = TestBed.runInInjectionContext(() =>
      authGuard({} as any, {} as any)
    );
    expect(result).toBe(true);
  });

  it('should redirect to login when not authenticated', () => {
    authServiceSpy.isAuthenticated.and.returnValue(false);
    routerSpy.createUrlTree.and.returnValue({} as any);
    TestBed.runInInjectionContext(() =>
      authGuard({} as any, {} as any)
    );
    expect(routerSpy.createUrlTree).toHaveBeenCalledWith(['/login']);
  });
});
```

## End-to-End Testing

Modern Ionic Angular projects use Cypress or Playwright for E2E testing.

### Cypress Setup

```bash
ng add @cypress/schematic
```

Run E2E tests:

```bash
ng e2e
```

### Playwright Setup

```bash
npm init playwright@latest
```

Run E2E tests:

```bash
npx playwright test
```

E2E tests verify complete user flows (login, form submission, navigation) against a running app instance. Use `data-testid` attributes on Ionic components for stable selectors:

```html
<ion-button data-testid="submit-btn" type="submit">Submit</ion-button>
```

```typescript
// Playwright example
await page.getByTestId('submit-btn').click();
```
