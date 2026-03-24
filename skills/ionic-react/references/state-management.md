# State Management in Ionic React

## Overview

Ionic React apps use standard React state management patterns. The choice depends on the project's complexity and existing tooling. This reference covers patterns specific to Ionic React, particularly around page caching behavior and overlay interactions.

## Page Caching and State

Because `IonRouterOutlet` keeps pages mounted in the DOM, state persists across navigation. This has implications:

- **State is preserved when navigating away and back.** Navigating from Page A to Page B and back to Page A does **not** reset Page A's state.
- **Use Ionic lifecycle hooks to refresh state.** If data should be refetched on every visit, use `useIonViewWillEnter` instead of `useEffect`.
- **Long-lived pages may hold stale data.** Combine Ionic lifecycle hooks with React state to ensure data stays current.

```typescript
import { useState } from 'react';
import { IonPage, IonContent, useIonViewWillEnter } from '@ionic/react';

const UserProfile: React.FC = () => {
  const [user, setUser] = useState<User | null>(null);

  // Refreshes on every page visit, not just initial mount.
  useIonViewWillEnter(() => {
    fetchCurrentUser().then(setUser);
  });

  return (
    <IonPage>
      <IonContent>
        {user && <p>{user.name}</p>}
      </IonContent>
    </IonPage>
  );
};
```

## React Context

For small to medium apps, React Context works well to share state across Ionic pages:

```typescript
// src/context/AuthContext.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface AuthContextType {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (credentials: Credentials) => {
    const result = await authService.login(credentials);
    setUser(result.user);
  }, []);

  const logout = useCallback(async () => {
    await authService.logout();
    setUser(null);
  }, []);

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

Place the provider **outside** `IonReactRouter` so all pages have access:

```typescript
const App: React.FC = () => (
  <IonApp>
    <AuthProvider>
      <IonReactRouter>
        <IonRouterOutlet>
          <Route exact path="/login" component={Login} />
          <Route exact path="/home" component={Home} />
        </IonRouterOutlet>
      </IonReactRouter>
    </AuthProvider>
  </IonApp>
);
```

## Redux Toolkit

For larger apps using Redux Toolkit, wrap the Ionic app in the Redux `Provider`:

```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import itemsReducer from './itemsSlice';

export const store = configureStore({
  reducer: {
    items: itemsReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```typescript
// src/App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

const App: React.FC = () => (
  <Provider store={store}>
    <IonApp>
      <IonReactRouter>
        <IonRouterOutlet>
          {/* routes */}
        </IonRouterOutlet>
      </IonReactRouter>
    </IonApp>
  </Provider>
);
```

Dispatch actions in Ionic lifecycle hooks to refresh data on page visits:

```typescript
import { useIonViewWillEnter } from '@ionic/react';
import { useAppDispatch, useAppSelector } from '../store';
import { fetchItems } from '../store/itemsSlice';

const ItemList: React.FC = () => {
  const dispatch = useAppDispatch();
  const items = useAppSelector((state) => state.items.list);

  useIonViewWillEnter(() => {
    dispatch(fetchItems());
  });

  // render items...
};
```

## Zustand

Zustand integrates with Ionic React without providers:

```typescript
// src/store/useItemStore.ts
import { create } from 'zustand';

interface ItemStore {
  items: Item[];
  loading: boolean;
  fetchItems: () => Promise<void>;
  addItem: (item: Item) => void;
}

export const useItemStore = create<ItemStore>((set) => ({
  items: [],
  loading: false,
  fetchItems: async () => {
    set({ loading: true });
    const items = await itemService.getAll();
    set({ items, loading: false });
  },
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));
```

Use in Ionic pages with lifecycle hooks:

```typescript
import { IonPage, IonContent, useIonViewWillEnter } from '@ionic/react';
import { useItemStore } from '../store/useItemStore';

const ItemList: React.FC = () => {
  const { items, loading, fetchItems } = useItemStore();

  useIonViewWillEnter(() => {
    fetchItems();
  });

  // render items...
};
```

## TanStack Query (React Query)

TanStack Query works with Ionic React for server state management. Use the `refetchOnWindowFocus` option combined with Ionic lifecycle hooks:

```typescript
// src/App.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

const App: React.FC = () => (
  <QueryClientProvider client={queryClient}>
    <IonApp>
      <IonReactRouter>
        <IonRouterOutlet>
          {/* routes */}
        </IonRouterOutlet>
      </IonReactRouter>
    </IonApp>
  </QueryClientProvider>
);
```

Invalidate queries on page enter to handle Ionic's page caching:

```typescript
import { useQueryClient, useQuery } from '@tanstack/react-query';
import { IonPage, IonContent, useIonViewWillEnter } from '@ionic/react';

const ItemList: React.FC = () => {
  const queryClient = useQueryClient();
  const { data: items, isLoading } = useQuery({
    queryKey: ['items'],
    queryFn: fetchItems,
  });

  useIonViewWillEnter(() => {
    queryClient.invalidateQueries({ queryKey: ['items'] });
  });

  // render items...
};
```

## Sharing State Between Pages and Overlays

Ionic modals and popovers are separate from the page component tree. To share state between a page and an overlay:

1. **Pass callbacks via props** when using `useIonModal` or inline modal components.
2. **Use global state** (context, Redux, Zustand) that both the page and the overlay can access.

```typescript
import { useState } from 'react';
import { IonButton, IonContent, IonModal, IonPage, IonInput, IonItem } from '@ionic/react';

const ParentPage: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  const [items, setItems] = useState<string[]>([]);

  const addItem = (name: string) => {
    setItems((prev) => [...prev, name]);
    setIsOpen(false);
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={() => setIsOpen(true)}>Add Item</IonButton>
        {items.map((item, i) => (
          <IonItem key={i}>{item}</IonItem>
        ))}

        <IonModal isOpen={isOpen} onDidDismiss={() => setIsOpen(false)}>
          <AddItemForm onAdd={addItem} onCancel={() => setIsOpen(false)} />
        </IonModal>
      </IonContent>
    </IonPage>
  );
};
```
