# Custom Hooks for Native Features

Reusable custom hooks that wrap Capacitor plugins and provide a React-idiomatic API.

## useNetwork

Monitor network connectivity:

```typescript
import { useEffect, useState } from 'react';
import { Network, ConnectionStatus } from '@capacitor/network';

export const useNetwork = () => {
  const [status, setStatus] = useState<ConnectionStatus>({
    connected: true,
    connectionType: 'unknown',
  });

  useEffect(() => {
    Network.getStatus().then(setStatus);

    const listener = Network.addListener('networkStatusChange', setStatus);

    return () => {
      listener.then(handle => handle.remove());
    };
  }, []);

  return status;
};
```

## useKeyboard

Track keyboard visibility on native platforms:

```typescript
import { useEffect, useState } from 'react';
import { Capacitor } from '@capacitor/core';
import { Keyboard } from '@capacitor/keyboard';

export const useKeyboard = () => {
  const [isOpen, setIsOpen] = useState(false);
  const [keyboardHeight, setKeyboardHeight] = useState(0);

  useEffect(() => {
    if (!Capacitor.isNativePlatform()) return;

    const showListener = Keyboard.addListener('keyboardWillShow', (info) => {
      setIsOpen(true);
      setKeyboardHeight(info.keyboardHeight);
    });

    const hideListener = Keyboard.addListener('keyboardWillHide', () => {
      setIsOpen(false);
      setKeyboardHeight(0);
    });

    return () => {
      showListener.then(handle => handle.remove());
      hideListener.then(handle => handle.remove());
    };
  }, []);

  return { isOpen, keyboardHeight };
};
```

## useAppState

Respond to app foreground/background transitions:

```typescript
import { useEffect, useRef, useState } from 'react';
import { App } from '@capacitor/app';

export const useAppState = () => {
  const [isActive, setIsActive] = useState(true);
  const onResumeRef = useRef<(() => void) | null>(null);
  const onPauseRef = useRef<(() => void) | null>(null);

  useEffect(() => {
    const resumeListener = App.addListener('resume', () => {
      setIsActive(true);
      onResumeRef.current?.();
    });

    const pauseListener = App.addListener('pause', () => {
      setIsActive(false);
      onPauseRef.current?.();
    });

    return () => {
      resumeListener.then(handle => handle.remove());
      pauseListener.then(handle => handle.remove());
    };
  }, []);

  const onResume = (callback: () => void) => {
    onResumeRef.current = callback;
  };

  const onPause = (callback: () => void) => {
    onPauseRef.current = callback;
  };

  return { isActive, onResume, onPause };
};
```

## useGeolocation

Get and watch the device position:

```typescript
import { useEffect, useState, useCallback } from 'react';
import { Geolocation, Position } from '@capacitor/geolocation';

export const useGeolocation = () => {
  const [position, setPosition] = useState<Position | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const getCurrentPosition = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const pos = await Geolocation.getCurrentPosition();
      setPosition(pos);
      return pos;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Failed to get position';
      setError(message);
      return null;
    } finally {
      setLoading(false);
    }
  }, []);

  return { position, error, loading, getCurrentPosition };
};
```

## useCamera

Capture photos with loading/error handling:

```typescript
import { useState, useCallback } from 'react';
import { Camera, CameraResultType, CameraSource, Photo } from '@capacitor/camera';

interface UseCameraOptions {
  quality?: number;
  resultType?: CameraResultType;
  source?: CameraSource;
}

export const useCamera = (options?: UseCameraOptions) => {
  const [photo, setPhoto] = useState<Photo | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const takePhoto = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await Camera.getPhoto({
        quality: options?.quality ?? 90,
        resultType: options?.resultType ?? CameraResultType.Uri,
        source: options?.source ?? CameraSource.Prompt,
      });
      setPhoto(result);
      return result;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Failed to take photo';
      setError(message);
      return null;
    } finally {
      setLoading(false);
    }
  }, [options?.quality, options?.resultType, options?.source]);

  return { photo, error, loading, takePhoto };
};
```

## usePreferences

Persist and retrieve key-value data:

```typescript
import { useEffect, useState, useCallback } from 'react';
import { Preferences } from '@capacitor/preferences';

export const usePreferences = <T>(key: string, defaultValue: T) => {
  const [value, setValue] = useState<T>(defaultValue);
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    Preferences.get({ key }).then(({ value: stored }) => {
      if (stored !== null) {
        setValue(JSON.parse(stored));
      }
      setLoaded(true);
    });
  }, [key]);

  const set = useCallback(
    async (newValue: T) => {
      setValue(newValue);
      await Preferences.set({ key, value: JSON.stringify(newValue) });
    },
    [key],
  );

  const remove = useCallback(async () => {
    setValue(defaultValue);
    await Preferences.remove({ key });
  }, [key, defaultValue]);

  return { value, set, remove, loaded };
};
```
