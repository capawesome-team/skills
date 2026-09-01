# MapLibre

Unofficial Capacitor plugin to create native [MapLibre](https://maplibre.org/) maps. Renders with the native MapLibre SDKs on Android and iOS and with MapLibre GL JS on the Web, and supports styles, camera control, markers, polylines, GeoJSON layers and user location.

**Package:** `@capawesome/capacitor-maplibre`
**Platforms:** Android, iOS, Web
**Capawesome Insiders:** No

## Installation

```bash
npm install @capawesome/capacitor-maplibre
npx cap sync
```

## Configuration

### Android

#### Variables

Defined in `android/variables.gradle`:

- `$mapLibreVersion` version of `org.maplibre.gl:android-sdk-vulkan-opengl` (default: `13.5.0`)
- `$mapLibreAnnotationVersion` version of `org.maplibre.gl:android-plugin-annotation` (default: `4.0.0`)

#### Permissions

Only required to display the location of the user. Add **before or after** the `application` tag in `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### iOS

#### Privacy Descriptions

Only required to display the location of the user. Add to `ios/App/App/Info.plist`:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app needs your location to display it on the map.</string>
```

Without this key, `checkPermissions()`, `requestPermissions()` and `enableUserLocation(...)` reject with an error.

### Web

Import the MapLibre GL JS stylesheet once in the app (for example in `src/main.ts`). It is not bundled with the plugin, and without it the map canvas and the markers are mispositioned:

```typescript
import 'maplibre-gl/dist/maplibre-gl.css';
```

## Usage

### Prepare the map element (required)

On Android and iOS the map is a native view rendered **behind** the web view. Add an **empty** element that defines the position and size of the map:

```html
<div id="map"></div>
```

```css
#map {
  /* The map must not have a background so that the native map stays visible. */
  background: transparent;
  height: 400px;
  width: 100%;
}
```

The map element **and every ancestor** that covers the map region (including `body`) must use `background: transparent`, otherwise the web view paints over the native map and the map is not visible. The element must stay empty — never render content into it. Elements that are not ancestors of the map element (floating action buttons, bottom sheets, dialogs) are displayed above the map as usual. The plugin keeps the native map in sync with the element position and size automatically, including while scrolling or resizing.

### Create a map

```typescript
import { MapLibre } from '@capawesome/capacitor-maplibre';

const createMap = async () => {
  await MapLibre.createMap({
    center: { latitude: 48.137154, longitude: 11.576124 },
    elementId: 'map',
    mapId: 'my-map',
    styleUrl: 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
    zoom: 12,
  });
};
```

### Move the camera

```typescript
import { MapLibre } from '@capawesome/capacitor-maplibre';

const moveCamera = async () => {
  await MapLibre.setCamera({
    animate: true,
    animationDuration: 1000,
    bearing: 30,
    center: { latitude: 52.520008, longitude: 13.404954 },
    mapId: 'my-map',
    pitch: 45,
    zoom: 11,
  });
  await MapLibre.fitBounds({
    animate: true,
    bounds: {
      northeast: { latitude: 55.058347, longitude: 15.041896 },
      southwest: { latitude: 47.270111, longitude: 5.866342 },
    },
    mapId: 'my-map',
    padding: { bottom: 32, left: 32, right: 32, top: 32 },
  });
};
```

### Add and update markers

```typescript
import { MapLibre, MarkerIconAnchor } from '@capawesome/capacitor-maplibre';

const addMarker = async () => {
  await MapLibre.addMarker({
    mapId: 'my-map',
    marker: {
      coordinates: { latitude: 48.137154, longitude: 11.576124 },
      iconAnchor: MarkerIconAnchor.Center,
      iconSize: { height: 32, width: 32 },
      iconUrl: 'https://example.com/marker.png',
      id: 'my-marker',
    },
  });
  await MapLibre.updateMarkerById({
    animate: true,
    animationDuration: 1000,
    coordinates: { latitude: 48.370545, longitude: 10.89779 },
    mapId: 'my-map',
    markerId: 'my-marker',
    rotation: 90,
  });
};
```

### Add a polyline

```typescript
import { MapLibre } from '@capawesome/capacitor-maplibre';

const addPolyline = async () => {
  await MapLibre.addPolyline({
    mapId: 'my-map',
    polyline: {
      color: '#3887be',
      coordinates: [
        { latitude: 48.137154, longitude: 11.576124 },
        { latitude: 49.45203, longitude: 11.076665 },
        { latitude: 52.520008, longitude: 13.404954 },
      ],
      id: 'my-polyline',
      width: 5,
    },
  });
};
```

### Add GeoJSON data

```typescript
import { LayerType, MapLibre } from '@capawesome/capacitor-maplibre';

const addGeoJson = async () => {
  await MapLibre.addGeoJsonSource({
    mapId: 'my-map',
    sourceId: 'my-source',
    url: 'https://example.com/routes.geojson',
  });
  await MapLibre.addLayer({
    layerId: 'my-layer',
    mapId: 'my-map',
    paint: { lineColor: '#3887be', lineWidth: 4 },
    sourceId: 'my-source',
    type: LayerType.Line,
  });
};
```

### Display the location of the user

```typescript
import { MapLibre, UserTrackingMode } from '@capawesome/capacitor-maplibre';

const enableUserLocation = async () => {
  let status = await MapLibre.checkPermissions();
  if (status.location === 'prompt') {
    status = await MapLibre.requestPermissions();
  }
  if (status.location !== 'granted') {
    return;
  }
  await MapLibre.enableUserLocation({
    mapId: 'my-map',
    trackingMode: UserTrackingMode.Follow,
  });
};
```

### Listen for events and destroy the map

```typescript
import { MapLibre } from '@capawesome/capacitor-maplibre';

const addListeners = async () => {
  await MapLibre.addListener('mapClick', event => {
    console.log('Map clicked:', event.coordinates);
  });
  await MapLibre.addListener('markerClick', event => {
    console.log('Marker clicked:', event.markerId);
  });
  await MapLibre.addListener('cameraIdle', event => {
    console.log('Camera idle:', event.camera);
  });
};

const destroyMap = async () => {
  await MapLibre.destroyMap({ mapId: 'my-map' });
};
```

## Notes

- No plugin configuration in `capacitor.config.ts` is required.
- Unlike `@capacitor/google-maps` (see `references/capacitor-google-maps.md`), this plugin requires no Google API key and no vendor account. It renders any open style that follows the MapLibre Style Spec — free options include CARTO basemaps, OpenFreeMap and Versatiles. Commercial providers such as MapTiler still require their own API key. Follow the attribution requirements of the chosen provider.
- If no style is provided, the MapLibre demo style (`https://demotiles.maplibre.org/style.json`) is used. It is a test style and must not be used in production.
- `createMap()` resolves as soon as the style has finished loading. Map methods target a map by its `mapId`, so multiple maps can be created at the same time.
- `setStyle()` loads a new style; all markers, polylines, sources and layers must be added again afterwards.
- `addGeoJsonSource()` and `updateGeoJsonSourceById()` accept exactly one of `data` and `url`. Layers using a source must be removed before the source itself.
- `LayerType` is `Circle`, `Fill` or `Line`. `LayerPaint` properties that do not apply to the layer type are ignored.
- Marker `iconUrl` must be an `https` URL or a data URI; without it a default pin icon is used. `iconAnchor` defaults to `MarkerIconAnchor.Bottom`.
- Draggable markers and the `markerDrag`, `markerDragStart` and `markerDragEnd` events are only available on Android and Web — iOS renders markers as symbol layers, which do not support native dragging.
- `UserTrackingMode` is `None`, `Follow`, `FollowWithCourse` or `FollowWithHeading`. On Web, `FollowWithCourse` and `FollowWithHeading` behave like `Follow`.
- Events: `cameraIdle`, `cameraMoveStarted`, `mapClick`, `markerClick`, `markerDrag`, `markerDragEnd`, `markerDragStart`, `userLocationChange`. The `elementFromPointRequest` event and the `elementFromPointResult()` and `setFrame()` methods are handled by the plugin internally and must not be used manually.
- Use `setGesturesEnabled()` to toggle `pan`, `zoom`, `rotate` and `tilt` at runtime; only the provided gestures are changed.
- The plugin does not work offline — styles, fonts, sprites and tiles are loaded from the network.
- Call `destroyMap()` to release the resources of a map.
