# Plugin Usage Patterns in React

## Direct Import and Use

Import Capacitor plugins as ES modules and call their methods in event handlers, effects, or custom hooks:

```typescript
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';

const PhotoButton: React.FC = () => {
  const takePhoto = async () => {
    const image = await Camera.getPhoto({
      quality: 90,
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
    });
    console.log('Photo URI:', image.webPath);
  };

  return <button onClick={takePhoto}>Take Photo</button>;
};
```

## Event Listeners in useEffect

Register Capacitor event listeners inside `useEffect` and clean up on unmount:

```typescript
import { useEffect, useState } from 'react';
import { Network, ConnectionStatus } from '@capacitor/network';

const NetworkStatus: React.FC = () => {
  const [status, setStatus] = useState<ConnectionStatus | null>(null);

  useEffect(() => {
    // Get initial status
    Network.getStatus().then(setStatus);

    // Listen for changes
    const listener = Network.addListener('networkStatusChange', (newStatus) => {
      setStatus(newStatus);
    });

    return () => {
      listener.then(handle => handle.remove());
    };
  }, []);

  if (!status) return null;

  return <p>Connected: {status.connected ? 'Yes' : 'No'}</p>;
};
```

## Platform Guards

Wrap platform-specific plugin calls with a platform check:

```typescript
import { Capacitor } from '@capacitor/core';
import { Haptics, ImpactStyle } from '@capacitor/haptics';

const triggerHaptic = async () => {
  if (Capacitor.isNativePlatform()) {
    await Haptics.impact({ style: ImpactStyle.Medium });
  }
};
```

## Async Operations with Loading State

Wrap plugin calls with loading and error state:

```typescript
import { useState } from 'react';
import { Geolocation, Position } from '@capacitor/geolocation';

const LocationButton: React.FC = () => {
  const [position, setPosition] = useState<Position | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const getLocation = async () => {
    setLoading(true);
    setError(null);
    try {
      const pos = await Geolocation.getCurrentPosition();
      setPosition(pos);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to get location');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <button onClick={getLocation} disabled={loading}>
        {loading ? 'Getting location...' : 'Get Location'}
      </button>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      {position && (
        <p>
          Lat: {position.coords.latitude}, Lng: {position.coords.longitude}
        </p>
      )}
    </div>
  );
};
```

## Permissions Pattern

Check and request permissions before using a plugin:

```typescript
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';

const useCamera = () => {
  const takePhoto = async () => {
    const permissions = await Camera.checkPermissions();

    if (permissions.camera === 'denied') {
      throw new Error('Camera permission was denied. Enable it in device settings.');
    }

    if (permissions.camera === 'prompt' || permissions.camera === 'prompt-with-rationale') {
      const requested = await Camera.requestPermissions({ permissions: ['camera'] });
      if (requested.camera === 'denied') {
        throw new Error('Camera permission was denied.');
      }
    }

    return Camera.getPhoto({
      quality: 90,
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
    });
  };

  return { takePhoto };
};
```

## Service Module Pattern

For complex plugin interactions, extract logic into a service module in `src/services/`:

```typescript
// src/services/storage.ts
import { Preferences } from '@capacitor/preferences';

export const StorageService = {
  async get<T>(key: string): Promise<T | null> {
    const { value } = await Preferences.get({ key });
    return value ? JSON.parse(value) : null;
  },

  async set<T>(key: string, value: T): Promise<void> {
    await Preferences.set({ key, value: JSON.stringify(value) });
  },

  async remove(key: string): Promise<void> {
    await Preferences.remove({ key });
  },

  async clear(): Promise<void> {
    await Preferences.clear();
  },
};
```

Then use it in components or hooks:

```typescript
import { useEffect, useState } from 'react';
import { StorageService } from '../services/storage';

const usePersistedState = <T>(key: string, defaultValue: T) => {
  const [value, setValue] = useState<T>(defaultValue);
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    StorageService.get<T>(key).then((stored) => {
      if (stored !== null) {
        setValue(stored);
      }
      setLoaded(true);
    });
  }, [key]);

  const setAndPersist = async (newValue: T) => {
    setValue(newValue);
    await StorageService.set(key, newValue);
  };

  return [value, setAndPersist, loaded] as const;
};
```
