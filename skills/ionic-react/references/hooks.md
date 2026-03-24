# Ionic React Hooks

## Overview

Ionic React provides framework-specific hooks that simplify interaction with Ionic controllers, overlays, and platform features. All hooks are imported from `@ionic/react`.

## Overlay Hooks

### useIonAlert

Present alert dialogs without managing state manually:

```typescript
import { IonButton, IonContent, IonPage, useIonAlert } from '@ionic/react';

const AlertExample: React.FC = () => {
  const [presentAlert] = useIonAlert();

  const showConfirm = async () => {
    await presentAlert({
      header: 'Confirm',
      message: 'Are you sure you want to delete this item?',
      buttons: [
        { text: 'Cancel', role: 'cancel' },
        {
          text: 'Delete',
          role: 'destructive',
          handler: () => {
            deleteItem();
          },
        },
      ],
    });
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={showConfirm}>Delete Item</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

### useIonToast

Show toast notifications:

```typescript
import { IonButton, IonContent, IonPage, useIonToast } from '@ionic/react';

const ToastExample: React.FC = () => {
  const [presentToast] = useIonToast();

  const showToast = async () => {
    await presentToast({
      message: 'Item saved successfully.',
      duration: 2000,
      position: 'bottom',
      color: 'success',
    });
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={showToast}>Save</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

### useIonActionSheet

Present action sheets:

```typescript
import { IonButton, IonContent, IonPage, useIonActionSheet } from '@ionic/react';

const ActionSheetExample: React.FC = () => {
  const [presentActionSheet] = useIonActionSheet();

  const showActions = async () => {
    await presentActionSheet({
      header: 'Options',
      buttons: [
        { text: 'Share', handler: () => shareItem() },
        { text: 'Edit', handler: () => editItem() },
        { text: 'Delete', role: 'destructive', handler: () => deleteItem() },
        { text: 'Cancel', role: 'cancel' },
      ],
    });
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={showActions}>Options</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

### useIonLoading

Show loading indicators:

```typescript
import { IonButton, IonContent, IonPage, useIonLoading } from '@ionic/react';

const LoadingExample: React.FC = () => {
  const [presentLoading, dismissLoading] = useIonLoading();

  const doWork = async () => {
    await presentLoading({ message: 'Saving...' });
    try {
      await saveData();
    } finally {
      await dismissLoading();
    }
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={doWork}>Save</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

### useIonModal

Present modals programmatically:

```typescript
import { IonButton, IonContent, IonPage, useIonModal } from '@ionic/react';

interface ModalContentProps {
  onDismiss: (data?: string) => void;
}

const ModalContent: React.FC<ModalContentProps> = ({ onDismiss }) => (
  <IonPage>
    <IonContent>
      <p>Modal content here</p>
      <IonButton onClick={() => onDismiss('confirmed')}>Confirm</IonButton>
      <IonButton onClick={() => onDismiss()}>Cancel</IonButton>
    </IonContent>
  </IonPage>
);

const ModalExample: React.FC = () => {
  const [present, dismiss] = useIonModal(ModalContent, {
    onDismiss: (data?: string) => {
      dismiss(data);
      if (data === 'confirmed') {
        handleConfirm();
      }
    },
  });

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={() => present()}>Open Modal</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

### useIonPicker

Present picker dialogs:

```typescript
import { IonButton, IonContent, IonPage, useIonPicker } from '@ionic/react';

const PickerExample: React.FC = () => {
  const [presentPicker] = useIonPicker();

  const showPicker = async () => {
    await presentPicker({
      columns: [
        {
          name: 'color',
          options: [
            { text: 'Red', value: 'red' },
            { text: 'Blue', value: 'blue' },
            { text: 'Green', value: 'green' },
          ],
        },
      ],
      buttons: [
        { text: 'Cancel', role: 'cancel' },
        {
          text: 'Confirm',
          handler: (selected) => {
            console.log('Selected:', selected.color.value);
          },
        },
      ],
    });
  };

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={showPicker}>Pick Color</IonButton>
      </IonContent>
    </IonPage>
  );
};
```

## Navigation Hook

### useIonRouter

Programmatic navigation with Ionic transition animations. See `references/routing.md` for full details.

```typescript
import { useIonRouter } from '@ionic/react';

const MyComponent: React.FC = () => {
  const router = useIonRouter();

  const navigate = () => {
    router.push('/detail/123', 'forward', 'push');
  };

  return <IonButton onClick={navigate}>Go</IonButton>;
};
```

## Platform Hook

### useIonPopover

Present popovers programmatically:

```typescript
import { IonButton, IonContent, IonList, IonItem, IonPage, useIonPopover } from '@ionic/react';

const PopoverContent: React.FC<{ onDismiss: () => void }> = ({ onDismiss }) => (
  <IonContent>
    <IonList>
      <IonItem button onClick={onDismiss}>Option 1</IonItem>
      <IonItem button onClick={onDismiss}>Option 2</IonItem>
    </IonList>
  </IonContent>
);

const PopoverExample: React.FC = () => {
  const [presentPopover, dismissPopover] = useIonPopover(PopoverContent, {
    onDismiss: () => dismissPopover(),
  });

  return (
    <IonPage>
      <IonContent>
        <IonButton onClick={(e) => presentPopover({ event: e.nativeEvent })}>
          Show Popover
        </IonButton>
      </IonContent>
    </IonPage>
  );
};
```

## Summary Table

| Hook                  | Purpose                                 |
| --------------------- | --------------------------------------- |
| `useIonAlert`         | Present alert dialogs                   |
| `useIonToast`         | Show toast notifications                |
| `useIonActionSheet`   | Present action sheets                   |
| `useIonLoading`       | Show loading indicators                 |
| `useIonModal`         | Present modals programmatically         |
| `useIonPicker`        | Present picker dialogs                  |
| `useIonPopover`       | Present popovers programmatically       |
| `useIonRouter`        | Programmatic navigation with animations |
| `useIonViewWillEnter` | Page lifecycle — before enter animation |
| `useIonViewDidEnter`  | Page lifecycle — after enter animation  |
| `useIonViewWillLeave` | Page lifecycle — before leave animation |
| `useIonViewDidLeave`  | Page lifecycle — after leave animation  |
