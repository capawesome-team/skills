# Ionic React Lifecycle Hooks

## Overview

Ionic React provides lifecycle hooks that fire when a page enters or leaves the view. These hooks are necessary because `IonRouterOutlet` keeps pages mounted in the DOM — standard React `useEffect` only fires on initial mount and final unmount, not on every page visit.

| Hook                    | Fires when                                            |
| ----------------------- | ----------------------------------------------------- |
| `useIonViewWillEnter`   | The page is about to enter the view (before animation) |
| `useIonViewDidEnter`    | The page has fully entered the view (after animation)  |
| `useIonViewWillLeave`   | The page is about to leave the view (before animation) |
| `useIonViewDidLeave`    | The page has fully left the view (after animation)     |

All hooks are imported from `@ionic/react`:

```typescript
import {
  useIonViewWillEnter,
  useIonViewDidEnter,
  useIonViewWillLeave,
  useIonViewDidLeave,
} from '@ionic/react';
```

## Requirements

These hooks only work inside components that are:
1. Rendered as the `component` of a `Route` inside `IonRouterOutlet`.
2. Wrapped in an `IonPage` component.

If the component is not a direct page component, the hooks will not fire. Child components of a page should receive data via props or context instead of using these hooks directly.

## Usage

### Refresh Data on Page Enter

Use `useIonViewWillEnter` to refresh data every time a page becomes visible:

```typescript
import { useState } from 'react';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonItem,
  IonLabel,
  useIonViewWillEnter,
} from '@ionic/react';

const ItemList: React.FC = () => {
  const [items, setItems] = useState<string[]>([]);

  useIonViewWillEnter(() => {
    // This runs every time the page enters the view, not just on first mount.
    fetchItems().then(setItems);
  });

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Items</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
        <IonList>
          {items.map((item, index) => (
            <IonItem key={index}>
              <IonLabel>{item}</IonLabel>
            </IonItem>
          ))}
        </IonList>
      </IonContent>
    </IonPage>
  );
};
```

### Cleanup on Page Leave

Use `useIonViewWillLeave` or `useIonViewDidLeave` to perform cleanup when navigating away:

```typescript
import { useRef } from 'react';
import {
  IonPage,
  IonContent,
  useIonViewWillLeave,
} from '@ionic/react';

const VideoPage: React.FC = () => {
  const videoRef = useRef<HTMLVideoElement>(null);

  useIonViewWillLeave(() => {
    // Pause video when leaving the page.
    videoRef.current?.pause();
  });

  return (
    <IonPage>
      <IonContent>
        <video ref={videoRef} src="/video.mp4" controls />
      </IonContent>
    </IonPage>
  );
};
```

### Combining Ionic and React Lifecycle

Use `useEffect` for one-time setup (e.g., initializing a service, creating a WebSocket connection) and Ionic lifecycle hooks for per-visit logic:

```typescript
import { useEffect, useState } from 'react';
import {
  IonPage,
  IonContent,
  useIonViewWillEnter,
  useIonViewDidLeave,
} from '@ionic/react';

const Dashboard: React.FC = () => {
  const [data, setData] = useState<DashboardData | null>(null);

  // Runs once on mount — use for one-time initialization.
  useEffect(() => {
    initializeAnalytics();
    return () => {
      cleanupAnalytics();
    };
  }, []);

  // Runs every time the page enters — use for data refresh.
  useIonViewWillEnter(() => {
    fetchDashboardData().then(setData);
  });

  // Runs every time the page leaves — use for pausing activity.
  useIonViewDidLeave(() => {
    pausePolling();
  });

  return (
    <IonPage>
      <IonContent>
        {data && <DashboardContent data={data} />}
      </IonContent>
    </IonPage>
  );
};
```

## Firing Order

When navigating from Page A to Page B:

1. `useIonViewWillLeave` (Page A)
2. `useIonViewWillEnter` (Page B)
3. Page transition animation runs
4. `useIonViewDidLeave` (Page A)
5. `useIonViewDidEnter` (Page B)

On initial app load (first page mount):

1. `useEffect` (React mount)
2. `useIonViewWillEnter`
3. `useIonViewDidEnter`

## Common Mistakes

- **Using `useEffect` for per-visit data fetching**: Since Ionic keeps pages in the DOM, `useEffect` with `[]` only runs once, not on every visit. Use `useIonViewWillEnter` instead.
- **Using lifecycle hooks in non-page components**: The hooks only fire in components directly rendered by a `Route` inside `IonRouterOutlet`. Extract the logic into the page component and pass data down.
- **Missing `IonPage` wrapper**: If the page component does not render `IonPage` as the root element, the lifecycle hooks will not fire.
