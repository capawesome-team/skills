# Routing in Ionic React

## Setup

Ionic React uses `@ionic/react-router`, which wraps React Router v5. Install both packages:

```bash
npm install @ionic/react-router react-router react-router-dom@5
```

For TypeScript projects, also install:

```bash
npm install -D @types/react-router @types/react-router-dom
```

## IonReactRouter

Replace the standard `BrowserRouter` with `IonReactRouter` in the root component. `IonReactRouter` is required for Ionic page transitions and navigation animations to work correctly.

```typescript
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';
import Home from './pages/Home';
import Detail from './pages/Detail';

setupIonicReact();

const App: React.FC = () => (
  <IonApp>
    <IonReactRouter>
      <IonRouterOutlet>
        <Route exact path="/home" component={Home} />
        <Route exact path="/detail/:id" component={Detail} />
        <Redirect exact from="/" to="/home" />
      </IonRouterOutlet>
    </IonReactRouter>
  </IonApp>
);
```

Key rules:
- `IonReactRouter` must be a direct child of `IonApp`.
- `IonRouterOutlet` must be a direct child of `IonReactRouter` (or inside `IonTabs`).
- Every `Route` inside `IonRouterOutlet` must use the `component` prop (not `render` or `children`), so that Ionic can manage the page stack correctly.

## IonRouterOutlet

`IonRouterOutlet` manages page transitions and keeps previous pages in the DOM for back-navigation animations. This differs from standard React Router, which unmounts pages on navigation.

```typescript
import { IonRouterOutlet } from '@ionic/react';
import { Route, Redirect } from 'react-router-dom';

<IonRouterOutlet>
  <Route exact path="/home" component={Home} />
  <Route exact path="/settings" component={Settings} />
  <Route exact path="/profile" component={Profile} />
  <Redirect exact from="/" to="/home" />
</IonRouterOutlet>
```

Because Ionic keeps pages mounted in the DOM:
- Components are **not unmounted** when navigating away. `useEffect` cleanup runs only when the component is truly removed from the stack.
- Use Ionic lifecycle hooks (`useIonViewWillEnter`, `useIonViewDidLeave`) instead of `useEffect` for logic that should run on every page visit. See `references/lifecycle.md`.

## Tab-Based Navigation

Use `IonTabs` with `IonTabBar` for tab navigation. Each tab has its own `IonRouterOutlet` for independent navigation stacks.

```typescript
import {
  IonApp,
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
  setupIonicReact,
} from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';
import { home, settings, person } from 'ionicons/icons';
import Home from './pages/Home';
import Settings from './pages/Settings';
import Profile from './pages/Profile';
import HomeDetail from './pages/HomeDetail';

setupIonicReact();

const App: React.FC = () => (
  <IonApp>
    <IonReactRouter>
      <IonTabs>
        <IonRouterOutlet>
          <Route exact path="/home" component={Home} />
          <Route exact path="/home/:id" component={HomeDetail} />
          <Route exact path="/settings" component={Settings} />
          <Route exact path="/profile" component={Profile} />
          <Redirect exact from="/" to="/home" />
        </IonRouterOutlet>
        <IonTabBar slot="bottom">
          <IonTabButton tab="home" href="/home">
            <IonIcon icon={home} />
            <IonLabel>Home</IonLabel>
          </IonTabButton>
          <IonTabButton tab="settings" href="/settings">
            <IonIcon icon={settings} />
            <IonLabel>Settings</IonLabel>
          </IonTabButton>
          <IonTabButton tab="profile" href="/profile">
            <IonIcon icon={person} />
            <IonLabel>Profile</IonLabel>
          </IonTabButton>
        </IonTabBar>
      </IonTabs>
    </IonReactRouter>
  </IonApp>
);
```

Key rules:
- All routes (including nested detail routes like `/home/:id`) must be placed inside the single `IonRouterOutlet` within `IonTabs`.
- The `tab` prop on `IonTabButton` must be a unique string.
- The `href` prop on `IonTabButton` must match the base route for that tab.
- Detail pages within a tab (e.g., `/home/:id`) keep the tab bar visible.

## Programmatic Navigation

Use the `useIonRouter` hook for programmatic navigation with Ionic transition animations:

```typescript
import { useIonRouter } from '@ionic/react';

const MyComponent: React.FC = () => {
  const router = useIonRouter();

  const goToDetail = (id: string) => {
    router.push(`/detail/${id}`, 'forward', 'push');
  };

  const goBack = () => {
    if (router.canGoBack()) {
      router.goBack();
    }
  };

  return (
    <>
      <IonButton onClick={() => goToDetail('123')}>View Detail</IonButton>
      <IonButton onClick={goBack}>Back</IonButton>
    </>
  );
};
```

The `useIonRouter` hook provides:
- `push(path, routerDirection?, routerAnimation?)` — navigate forward.
- `goBack()` — navigate back with animation.
- `canGoBack()` — check if back navigation is possible.
- `routeInfo` — current route information.

The `routerDirection` parameter controls the animation direction: `'forward'`, `'back'`, or `'root'`.

Alternatively, use `routerLink` on Ionic components for declarative navigation:

```typescript
<IonItem routerLink="/detail/123" routerDirection="forward">
  <IonLabel>Go to Detail</IonLabel>
</IonItem>

<IonButton routerLink="/settings" routerDirection="root">
  Settings
</IonButton>
```

## Route Guards

Implement route guards using React Router's `Redirect` or a wrapper component:

```typescript
import { Route, Redirect, RouteProps } from 'react-router-dom';

interface ProtectedRouteProps extends RouteProps {
  isAuthenticated: boolean;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  isAuthenticated,
  component: Component,
  ...rest
}) => (
  <Route
    {...rest}
    render={(props) =>
      isAuthenticated ? (
        <Component {...props} />
      ) : (
        <Redirect to="/login" />
      )
    }
  />
);
```

Usage inside `IonRouterOutlet`:

```typescript
<IonRouterOutlet>
  <Route exact path="/login" component={Login} />
  <ProtectedRoute
    exact
    path="/dashboard"
    component={Dashboard}
    isAuthenticated={isLoggedIn}
  />
</IonRouterOutlet>
```

## Route Parameters

Access route parameters using React Router hooks:

```typescript
import { useParams } from 'react-router-dom';

interface DetailParams {
  id: string;
}

const Detail: React.FC = () => {
  const { id } = useParams<DetailParams>();

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonButtons slot="start">
            <IonBackButton defaultHref="/home" />
          </IonButtons>
          <IonTitle>Detail {id}</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
        <p>Viewing item {id}</p>
      </IonContent>
    </IonPage>
  );
};
```

## Side Menu Navigation

Use `IonMenu` with `IonRouterOutlet` for side-menu navigation:

```typescript
import {
  IonApp,
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonMenu,
  IonMenuToggle,
  IonRouterOutlet,
  IonTitle,
  IonToolbar,
  setupIonicReact,
} from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';

setupIonicReact();

const App: React.FC = () => (
  <IonApp>
    <IonReactRouter>
      <IonMenu contentId="main-content">
        <IonHeader>
          <IonToolbar>
            <IonTitle>Menu</IonTitle>
          </IonToolbar>
        </IonHeader>
        <IonContent>
          <IonList>
            <IonMenuToggle>
              <IonItem routerLink="/home" routerDirection="root">
                <IonLabel>Home</IonLabel>
              </IonItem>
            </IonMenuToggle>
            <IonMenuToggle>
              <IonItem routerLink="/settings" routerDirection="root">
                <IonLabel>Settings</IonLabel>
              </IonItem>
            </IonMenuToggle>
          </IonList>
        </IonContent>
      </IonMenu>
      <IonRouterOutlet id="main-content">
        <Route exact path="/home" component={Home} />
        <Route exact path="/settings" component={Settings} />
        <Redirect exact from="/" to="/home" />
      </IonRouterOutlet>
    </IonReactRouter>
  </IonApp>
);
```

Key rules:
- The `contentId` prop on `IonMenu` must match the `id` prop on `IonRouterOutlet`.
- Wrap menu items in `IonMenuToggle` to auto-close the menu after navigation.
- Use `routerDirection="root"` for top-level menu navigation to reset the page stack.
