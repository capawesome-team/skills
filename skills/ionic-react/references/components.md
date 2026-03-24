# React-Specific Ionic Component Patterns

## Page Structure

Every Ionic React page must be wrapped in `IonPage`. This is required for Ionic transitions and lifecycle hooks to work.

```typescript
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButtons,
  IonBackButton,
} from '@ionic/react';

const DetailPage: React.FC = () => (
  <IonPage>
    <IonHeader>
      <IonToolbar>
        <IonButtons slot="start">
          <IonBackButton defaultHref="/home" />
        </IonButtons>
        <IonTitle>Detail</IonTitle>
      </IonToolbar>
    </IonHeader>
    <IonContent className="ion-padding">
      <p>Page content here</p>
    </IonContent>
  </IonPage>
);
```

Key rules:
- `IonPage` must be the **root element** of every page component.
- `IonHeader` is optional but recommended for pages with a toolbar.
- `IonContent` is required for scrollable content and pull-to-refresh.
- `IonBackButton` provides automatic back navigation with the correct animation. Set `defaultHref` as a fallback when there is no navigation history.

## Overlays as React Components

Ionic overlays (modals, popovers, action sheets) can be used as inline React components with `isOpen` state binding:

### Inline Modal

```typescript
import { useState } from 'react';
import {
  IonButton,
  IonButtons,
  IonContent,
  IonHeader,
  IonModal,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';

const ModalPage: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={() => setIsOpen(true)}>Open Modal</IonButton>

        <IonModal isOpen={isOpen} onDidDismiss={() => setIsOpen(false)}>
          <IonHeader>
            <IonToolbar>
              <IonTitle>Modal</IonTitle>
              <IonButtons slot="end">
                <IonButton onClick={() => setIsOpen(false)}>Close</IonButton>
              </IonButtons>
            </IonToolbar>
          </IonHeader>
          <IonContent className="ion-padding">
            <p>Modal content</p>
          </IonContent>
        </IonModal>
      </IonContent>
    </IonPage>
  );
};
```

### Sheet Modal

```typescript
<IonModal
  isOpen={isOpen}
  onDidDismiss={() => setIsOpen(false)}
  initialBreakpoint={0.5}
  breakpoints={[0, 0.25, 0.5, 0.75, 1]}
>
  <IonContent className="ion-padding">
    <p>Sheet modal content</p>
  </IonContent>
</IonModal>
```

### Inline Popover

```typescript
import { useState } from 'react';
import {
  IonButton,
  IonContent,
  IonItem,
  IonList,
  IonPage,
  IonPopover,
} from '@ionic/react';

const PopoverPage: React.FC = () => {
  const [popoverEvent, setPopoverEvent] = useState<Event | undefined>(undefined);
  const [isOpen, setIsOpen] = useState(false);

  return (
    <IonPage>
      <IonContent>
        <IonButton
          onClick={(e) => {
            setPopoverEvent(e.nativeEvent);
            setIsOpen(true);
          }}
        >
          Show Popover
        </IonButton>

        <IonPopover
          isOpen={isOpen}
          event={popoverEvent}
          onDidDismiss={() => setIsOpen(false)}
        >
          <IonContent>
            <IonList>
              <IonItem button onClick={() => setIsOpen(false)}>Option 1</IonItem>
              <IonItem button onClick={() => setIsOpen(false)}>Option 2</IonItem>
            </IonList>
          </IonContent>
        </IonPopover>
      </IonContent>
    </IonPage>
  );
};
```

## Lists with Virtual Scrolling

Use `IonList` with React rendering patterns for efficient lists. For very large lists, use `IonContent`'s built-in scroll features:

```typescript
import {
  IonContent,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
} from '@ionic/react';

interface Item {
  id: string;
  name: string;
}

interface ItemListProps {
  items: Item[];
}

const ItemList: React.FC<ItemListProps> = ({ items }) => (
  <IonPage>
    <IonContent>
      <IonList>
        {items.map((item) => (
          <IonItem key={item.id} routerLink={`/items/${item.id}`}>
            <IonLabel>{item.name}</IonLabel>
          </IonItem>
        ))}
      </IonList>
    </IonContent>
  </IonPage>
);
```

## Pull-to-Refresh

```typescript
import { useState } from 'react';
import {
  IonContent,
  IonPage,
  IonRefresher,
  IonRefresherContent,
} from '@ionic/react';

const RefreshPage: React.FC = () => {
  const [data, setData] = useState<string[]>([]);

  const handleRefresh = async (event: CustomEvent) => {
    const newData = await fetchData();
    setData(newData);
    event.detail.complete();
  };

  return (
    <IonPage>
      <IonContent>
        <IonRefresher slot="fixed" onIonRefresh={handleRefresh}>
          <IonRefresherContent />
        </IonRefresher>
        {data.map((item, i) => (
          <p key={i}>{item}</p>
        ))}
      </IonContent>
    </IonPage>
  );
};
```

## Infinite Scroll

```typescript
import { useState } from 'react';
import {
  IonContent,
  IonInfiniteScroll,
  IonInfiniteScrollContent,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
} from '@ionic/react';

const InfiniteScrollPage: React.FC = () => {
  const [items, setItems] = useState<string[]>(generateItems(0, 20));
  const [hasMore, setHasMore] = useState(true);

  const loadMore = async (event: CustomEvent) => {
    const newItems = await fetchMore(items.length, 20);
    setItems((prev) => [...prev, ...newItems]);
    if (newItems.length < 20) {
      setHasMore(false);
    }
    (event.target as HTMLIonInfiniteScrollElement).complete();
  };

  return (
    <IonPage>
      <IonContent>
        <IonList>
          {items.map((item, i) => (
            <IonItem key={i}>
              <IonLabel>{item}</IonLabel>
            </IonItem>
          ))}
        </IonList>
        <IonInfiniteScroll disabled={!hasMore} onIonInfinite={loadMore}>
          <IonInfiniteScrollContent loadingText="Loading more..." />
        </IonInfiniteScroll>
      </IonContent>
    </IonPage>
  );
};
```

## Forms with Ionic Components

Ionic form components emit `ionChange` events. Use the `onIonChange` prop:

```typescript
import { useState } from 'react';
import {
  IonButton,
  IonContent,
  IonInput,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonSelect,
  IonSelectOption,
  IonToggle,
} from '@ionic/react';

const FormPage: React.FC = () => {
  const [name, setName] = useState('');
  const [role, setRole] = useState<string>('');
  const [notifications, setNotifications] = useState(false);

  const handleSubmit = () => {
    console.log({ name, role, notifications });
  };

  return (
    <IonPage>
      <IonContent className="ion-padding">
        <IonList>
          <IonItem>
            <IonInput
              label="Name"
              labelPlacement="stacked"
              value={name}
              onIonInput={(e) => setName(e.detail.value ?? '')}
            />
          </IonItem>
          <IonItem>
            <IonSelect
              label="Role"
              labelPlacement="stacked"
              value={role}
              onIonChange={(e) => setRole(e.detail.value)}
            >
              <IonSelectOption value="admin">Admin</IonSelectOption>
              <IonSelectOption value="user">User</IonSelectOption>
            </IonSelect>
          </IonItem>
          <IonItem>
            <IonToggle
              checked={notifications}
              onIonChange={(e) => setNotifications(e.detail.checked)}
            >
              Enable Notifications
            </IonToggle>
          </IonItem>
        </IonList>
        <IonButton expand="block" onClick={handleSubmit}>
          Submit
        </IonButton>
      </IonContent>
    </IonPage>
  );
};
```

Key rules:
- Use `onIonInput` for `IonInput` and `IonTextarea` to get the value as the user types.
- Use `onIonChange` for `IonSelect`, `IonToggle`, `IonCheckbox`, `IonRadioGroup`, and `IonRange`.
- Access the value via `e.detail.value` (or `e.detail.checked` for toggles/checkboxes).
