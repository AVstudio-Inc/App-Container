# App Container

**Run any HTML5 app as a native mobile experience — no publishing required.**

[![App Store](https://img.shields.io/badge/App_Store-iOS-black?logo=apple)](https://apps.apple.com/us/app/avstudio-app-container/id6757149515)
[![Google Play](https://img.shields.io/badge/Google_Play-Android-green?logo=google-play)](https://play.google.com/store/apps/details?id=com.appcontainer.app)


https://apps.apple.com/us/app/avstudio-app-container/id6757149515
https://play.google.com/store/apps/details?id=com.appcontainer.app
---

App Container is a mobile shell for iOS and Android that turns HTML5 web applications into full-screen, native-feeling experiences. Install it once, load any number of HTML5 projects, and launch them as if they were standalone apps — all running privately and locally on the device without any cloud dependency.

---

## Why App Container

Publishing a dedicated mobile app for every HTML5 control interface is slow and expensive. App Container removes that barrier: package the project as a `.zip` or `.ch5z` archive, load it into the app, and it runs immediately — full-screen, offline-capable, with no browser chrome.

It is particularly suited for:

- **Building automation and smart home panels** — run Crestron, KNX, or custom HTML5 interfaces on a dedicated tablet
- **Industrial and commercial dashboards** — deploy HTML5 UIs to floor-mounted or wall-mounted devices
- **HTML5 developers** — test projects on real hardware without a development environment

---

## Features

### Project Management
Load HTML5 projects from a local file (`.zip`, `.ch5z`), a direct URL, or the built-in demo library. Multiple projects can be installed simultaneously; switching between them is instant.

On iOS, `.ch5z` files are registered as a system file type — projects can be sent directly from Files or any Share Sheet.

### Full-Fidelity Web Rendering
The embedded WebView runs with full JavaScript support, unrestricted media autoplay, and no cross-origin restrictions for local content.

- Self-signed and expired SSL certificates are accepted — essential for local network deployments
- `fetch` and `XMLHttpRequest` calls to external hosts are transparently proxied through the on-device server, resolving mixed-content and CORS issues with no changes to the project

### Remote Management API
When enabled, a management interface is available at `http://{device-ip}:8080/manage` — accessible from any browser on the same local network.

- Switch the active project remotely
- Install and remove projects
- Capture a live screenshot of the current screen

Access is secured with a Bearer token shown in the app settings.

### Developer Console
A built-in debugging environment designed for HTML5 developers:

- **Console** — captures `console.log/warn/error/info`, uncaught exceptions, and unhandled promise rejections, filterable by level with millisecond timestamps
- **Network** — monitors all `fetch` and `XMLHttpRequest` activity: method, URL, status, response time, and failed resource loads

### URL Parameter Management
Define key/value parameters that are appended to every project URL automatically — globally or per-project. Use this to inject tokens, server addresses, or environment flags without modifying project files.

### Display Controls
Fine-grained control over the device UI:

| Setting | Options |
|---|---|
| Status bar | Visible / Hidden / Extended behind |
| Navigation bar (Android) | Visible / Hidden |
| Full immersive mode | Hides both bars |
| Screen orientation | Auto / Portrait / Landscape |
| Keep screen awake | On / Off |
| Settings button visibility | Visible / Hidden |

On iOS, all settings are also accessible from the native iOS Settings app.

### Kiosk Mode
Lock the device to App Container for unattended deployments.

- **Android** — activates Screen Pinning via `startLockTask()`; optional PIN protection via system security settings
- **iOS** — guided setup for Guided Access with a direct link to Accessibility Settings

---

## Streaming API for Web Developers

---

### Native Player — `appcontainer.streamBridge`

The app exposes a JavaScript API at `window.appcontainer.streamBridge`. Posting a JSON message to it
creates (or removes) a native video overlay rendered above the WebView at
the exact position of a DOM element.

#### Opening a stream

```js
window.appcontainer.streamBridge.postMessage(JSON.stringify({
  action: 'open',
  id:     'cam1',          // unique identifier, arbitrary string
  url:    'rtsp://192.168.1.10:554/stream',
  divId:  'player-div',   // id of a placeholder <div> in your page
}));
```

The app reads the bounding rect of `player-div` and places the video surface over it.
The `<div>` itself is never touched — style it as a visible placeholder
while the stream initializes (dark background, spinner, etc.).

You can also specify the overlay position in fractional screen coordinates instead of a `divId`:

```js
window.appcontainer.streamBridge.postMessage(JSON.stringify({
  action: 'open',
  id:     'cam1',
  url:    'rtsp://...',
  rect:   { x: 0, y: 0, w: 1, h: 0.5 },  // fractions of screen width/height
}));
```

#### Updating position after scroll or resize

The overlay does **not** follow DOM scroll automatically. Call `resize` whenever the page scrolls
or the layout changes:

```js
window.appcontainer.streamBridge.postMessage(JSON.stringify({
  action: 'resize',
  id:     'cam1',
  divId:  'player-div',   // or rect: { x,y,w,h }
}));

// Typical wiring:
window.addEventListener('scroll',  () => resizeAll(), { passive: true });
window.addEventListener('resize',  () => resizeAll());
```

#### Closing a stream

```js
// Close one:
window.appcontainer.streamBridge.postMessage(JSON.stringify({ action: 'close', id: 'cam1' }));

// Close all:
window.appcontainer.streamBridge.postMessage(JSON.stringify({ action: 'closeAll' }));
```

#### Multiple simultaneous streams

Each stream requires a unique `id`. There is no built-in limit on the number of simultaneous streams,
though device hardware imposes practical limits depending on resolution and codec.

```js
['cam1', 'cam2', 'cam3', 'cam4'].forEach((id, i) => {
  window.appcontainer.streamBridge.postMessage(JSON.stringify({
    action: 'open',
    id,
    url:   `rtsp://192.168.1.${10 + i}:554/stream`,
    divId: `div-${id}`,
  }));
});
```

#### Streams are cleared on navigation

When the WebView navigates to a new page, all open overlays are closed automatically. Re-open them
in a `pageshow` or `DOMContentLoaded` handler if needed.

---

## Download

| Platform | Link |
|---|---|
| iOS (iPhone & iPad) | [App Store](https://apps.apple.com/ru/app/avstudio-app-container/id6757149515) |
| Android (phone & tablet) | [Google Play](https://play.google.com/store/apps/details?id=com.appcontainer.app) |

Both platforms are functionally equivalent. A 14-day free trial is available on first install — no account required.

---

## Subscription

After the trial, the full feature set (including the Remote API) requires an active subscription:

| Plan | Type |
|---|---|
| Monthly | Recurring |
| Annual | Recurring |
| Lifetime | One-time purchase |

Purchases are managed through the App Store and Google Play. Existing purchases can be restored at any time.

---
