# Ionic Angular Services and State Management

Patterns for managing state and business logic in Ionic Angular applications using Angular services.

## Service Basics

Services in Ionic Angular follow standard Angular patterns. Use `@Injectable({ providedIn: 'root' })` for singleton services available app-wide:

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class DataService {
  private items: Item[] = [];

  getAll(): Item[] {
    return this.items;
  }

  add(item: Item): void {
    this.items.push(item);
  }
}
```

## HTTP Data Fetching

### Setup

**Standalone apps** — provide `HttpClient` in `app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { provideIonicAngular } from '@ionic/angular/standalone';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes), provideIonicAngular({}), provideHttpClient()],
};
```

**NgModule apps** — import `HttpClientModule` in `app.module.ts`:

```typescript
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [BrowserModule, HttpClientModule, IonicModule.forRoot(), AppRoutingModule],
  // ...
})
export class AppModule {}
```

### Service with HTTP

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Item {
  id: number;
  name: string;
}

@Injectable({
  providedIn: 'root',
})
export class ItemService {
  private http = inject(HttpClient);
  private apiUrl = 'https://api.example.com/items';

  getAll(): Observable<Item[]> {
    return this.http.get<Item[]>(this.apiUrl);
  }

  getById(id: number): Observable<Item> {
    return this.http.get<Item>(`${this.apiUrl}/${id}`);
  }

  create(item: Omit<Item, 'id'>): Observable<Item> {
    return this.http.post<Item>(this.apiUrl, item);
  }

  update(id: number, item: Partial<Item>): Observable<Item> {
    return this.http.put<Item>(`${this.apiUrl}/${id}`, item);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

### Using the Service in a Page

```typescript
import { Component, inject } from '@angular/core';
import { ViewWillEnter } from '@ionic/angular';
import { ItemService, Item } from '../services/item.service';

@Component({ /* ... */ })
export class ItemListPage implements ViewWillEnter {
  private itemService = inject(ItemService);
  items: Item[] = [];
  loading = false;
  error: string | null = null;

  ionViewWillEnter() {
    this.loadItems();
  }

  private loadItems() {
    this.loading = true;
    this.error = null;
    this.itemService.getAll().subscribe({
      next: (items) => {
        this.items = items;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Failed to load items.';
        this.loading = false;
      },
    });
  }
}
```

Use `ionViewWillEnter` instead of `ngOnInit` to refresh data on every page visit (Ionic caches pages in the DOM).

## Reactive State with BehaviorSubject

For shared state that multiple components need to observe:

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

export interface UserProfile {
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root',
})
export class UserService {
  private userSubject = new BehaviorSubject<UserProfile | null>(null);

  get user$(): Observable<UserProfile | null> {
    return this.userSubject.asObservable();
  }

  get currentUser(): UserProfile | null {
    return this.userSubject.value;
  }

  setUser(user: UserProfile): void {
    this.userSubject.next(user);
  }

  clearUser(): void {
    this.userSubject.next(null);
  }
}
```

Consume in a component:

```typescript
import { Component, inject } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { UserService } from '../services/user.service';

@Component({
  selector: 'app-header',
  template: `
    @if (userService.user$ | async; as user) {
      <p>Welcome, {{ user.name }}</p>
    }
  `,
  standalone: true,
  imports: [AsyncPipe],
})
export class HeaderComponent {
  userService = inject(UserService);
}
```

## Angular Signals (Angular 16+)

For Angular 16+, use signals for reactive state as a simpler alternative to RxJS:

```typescript
import { Injectable, signal, computed } from '@angular/core';

export interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Injectable({
  providedIn: 'root',
})
export class CartService {
  private items = signal<CartItem[]>([]);

  readonly cartItems = this.items.asReadonly();
  readonly totalPrice = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  readonly itemCount = computed(() =>
    this.items().reduce((sum, item) => sum + item.quantity, 0)
  );

  addItem(item: Omit<CartItem, 'quantity'>): void {
    this.items.update((items) => {
      const existing = items.find((i) => i.id === item.id);
      if (existing) {
        return items.map((i) => (i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i));
      }
      return [...items, { ...item, quantity: 1 }];
    });
  }

  removeItem(id: number): void {
    this.items.update((items) => items.filter((i) => i.id !== id));
  }
}
```

Consume in a component:

```typescript
import { Component, inject } from '@angular/core';
import { CartService } from '../services/cart.service';

@Component({
  selector: 'app-cart',
  template: `
    <p>Items: {{ cartService.itemCount() }}</p>
    <p>Total: {{ cartService.totalPrice() | currency }}</p>
  `,
  standalone: true,
})
export class CartComponent {
  cartService = inject(CartService);
}
```

## Loading State Pattern with Ionic

Show loading indicators while fetching data:

```typescript
import { Component, inject } from '@angular/core';
import { LoadingController } from '@ionic/angular';
import { ItemService } from '../services/item.service';

@Component({ /* ... */ })
export class ItemListPage {
  private loadingCtrl = inject(LoadingController);
  private itemService = inject(ItemService);

  async loadItems() {
    const loading = await this.loadingCtrl.create({
      message: 'Loading items...',
    });
    await loading.present();

    this.itemService.getAll().subscribe({
      next: (items) => {
        this.items = items;
        loading.dismiss();
      },
      error: () => {
        loading.dismiss();
        // Show error toast
      },
    });
  }
}
```

For standalone, import `LoadingController` from `@ionic/angular/standalone`.

## Error Handling with Toast

```typescript
import { Component, inject } from '@angular/core';
import { ToastController } from '@ionic/angular';

@Component({ /* ... */ })
export class BasePage {
  private toastCtrl = inject(ToastController);

  async showError(message: string) {
    const toast = await this.toastCtrl.create({
      message,
      duration: 3000,
      color: 'danger',
      position: 'bottom',
    });
    await toast.present();
  }

  async showSuccess(message: string) {
    const toast = await this.toastCtrl.create({
      message,
      duration: 2000,
      color: 'success',
      position: 'bottom',
    });
    await toast.present();
  }
}
```

For standalone, import `ToastController` from `@ionic/angular/standalone`.

## Environment-Based Configuration

Use Angular's environment files for API URLs and feature flags:

`src/environments/environment.ts`:

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
};
```

`src/environments/environment.prod.ts`:

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.example.com',
};
```

Use in services:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../environments/environment';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);
  private baseUrl = environment.apiUrl;

  get<T>(path: string) {
    return this.http.get<T>(`${this.baseUrl}${path}`);
  }
}
```
