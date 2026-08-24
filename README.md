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
- On supported AVstudio panels: hardware control — terminal, GPIO, relays, LED, serial (RS-232/RS-485), UDP, and scheduled sleep/wake
- **AVS-10 / AVS-15 touch panels only**: WebRTC/WHEP proxy — bridge RTSP camera streams to browser-compatible WebRTC

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

#### Technician Access (AVS-10 / AVS-15 touch panels only)
On [AVS touch panels](https://avstudio.app/products/avs-touch-panel/), a hidden technician panel is available for on-site configuration — brightness, volume, network, PIN, installed apps, reboot — without leaving kiosk mode:

- While App Container itself is the active kiosk app, hold five fingers on the screen for five seconds, then enter the technician PIN.
- When a third-party app is pinned as the kiosk app instead, technician access is available remotely from the Remote Management API dashboard's Kiosk App control.

Not available on standard smartphones/tablets running the general App Container app.

---

## Streaming API for Web Developers

App Container offers two ways to display video streams:

- **Native Player** (`appcontainer.streamBridge`) — a Flutter-native overlay using
  libmpv/FFmpeg. Best for local kiosk screens: zero added latency, hardware decoding.
- **WebRTC / WHEP** — bridge RTSP to WebRTC for viewing in any browser. Ideal for
  remote dashboards. No plugin required.

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

Use `ResizeObserver` on the placeholder `<div>` — it fires on any layout change
(scroll, resize, font change, DOM mutation). Debounce to ~60 fps:

```js
const box = document.getElementById('player-div');
const ro = new ResizeObserver(() => {
  window.appcontainer.streamBridge.postMessage(JSON.stringify({
    action: 'resize',
    id:     'cam1',
    divId:  'player-div',
  }));
});
ro.observe(box);
```

No manual `resize()` calls needed — the observer handles everything automatically.

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

## Remote Hardware API

Hardware control features — terminal shell, GPIO & relay pins, LED indicator,
RS-232/RS-485 serial, UDP send, and scheduled sleep/wake — are available on
[AVS-10 / AVS-15 touch panels](https://avstudio.app/products/avs-touch-panel/).
Standard smartphones and tablets do not include these capabilities.

Endpoints are served at
`http://{device-ip}:8080/api/remote/`. All endpoints require a Bearer token
(shown in app settings).

### Terminal

Execute shell commands on the device. Commands run with a 30-second timeout.

```
POST /api/remote/terminal/exec
Content-Type: application/json
Authorization: Bearer <token>

{ "command": "cat /proc/version" }
```

Response:
```json
{
  "command": "cat /proc/version",
  "stdout": "Linux version 5.10.198...",
  "stderr": "",
  "exitCode": 0,
  "duration": "12ms"
}
```

### RTC Sleep / Wake

Schedule automatic device sleep and wake-up. The device goes to sleep at
`offTime` and wakes at `onTime`. Alarms persist across reboots.

```
GET  /api/remote/rtc/alarm
POST /api/remote/rtc/alarm
```

```json
{
  "enabled": true,
  "onTime":  "08:00",
  "offTime": "22:00",
  "onDays":  []
}
```

`onDays` — array of days (0=Sunday … 6=Saturday). Empty = every day.

### LED Control

Built-in LED strip control with RGB color support. State persists across reboots.

| Method | Endpoint | Body |
|--------|----------|------|
| `GET` | `/api/remote/led/state` | — |
| `POST` | `/api/remote/led/on` | `{ "on": true }` |
| `POST` | `/api/remote/led/color` | `{ "red": 255, "green": 128, "blue": 0 }` |

Response always includes full LED state:
```json
{
  "status": "ok",
  "state": { "on": true, "red": 255, "green": 128, "blue": 0 }
}
```

### GPIO & Relay

Four pins available via `/dev/gpio_control`:

| Pin | Label | Type |
|---|---|---|
| 0 | IO 1 | Digital I/O — read/write |
| 1 | IO 2 | Digital I/O — read/write |
| 2 | Relay 1 | Relay output (CLOSED/OPEN) |
| 3 | Relay 2 | Relay output (CLOSED/OPEN) |

```
POST /api/remote/gpio/write   { "pin": 2, "level": 1 }
POST /api/remote/gpio/read    { "pin": 0 }
```

Response:
```json
{ "status": "ok", "pin": 2, "level": 1, "ok": true }
```

### RS-232 / RS-485 Serial

Send and receive raw data over UART serial ports.

| Port | Interface |
|---|---|
| `/dev/ttyS5` | RS-232 |
| `/dev/ttyS9` | RS-485 |

```
POST /api/remote/serial/write  { "port": "/dev/ttyS5", "baudRate": 9600, "hex": "010600660001A815" }
POST /api/remote/serial/read   { "port": "/dev/ttyS9", "baudRate": 9600, "timeout": 500 }
POST /api/remote/serial/close  { "port": "/dev/ttyS5" }
```

Write response:
```json
{ "status": "ok", "port": "/dev/ttyS5", "sent": 8 }
```

Read response:
```json
{ "port": "/dev/ttyS5", "hex": "0103020000", "data": "..." }
```

### Power

Reboot or shut down the device remotely.

```
POST /api/remote/reboot       → restarts the device
POST /api/remote/poweroff      → shuts down the device
```

Returns `{ "status": "rebooting" }` or `{ "status": "powering off" }` before
the device shuts down.

### UDP Send

Send raw UDP datagrams — including broadcast — to any host on the local network.

```
POST /api/remote/udp/send
{ "host": "192.168.1.255", "port": 502, "hex": "010600660001A815" }
```

Response:
```json
{ "status": "ok", "host": "192.168.1.255", "port": 502, "sent": 8 }
```

Broadcast addresses (`.255` or `255.255.255.255`) are automatically detected
and sent via `SO_BROADCAST`.

### App Management (AVS-10 / AVS-15 touch panels only)

Install, list, and launch third-party Android apps — directly from the web
project, the Remote API, or programmatically from Flutter.

| Method | Endpoint | Body |
|--------|----------|------|
| `GET` | `/api/remote/apps/system` | — |
| `POST` | `/api/remote/apps/launch` | `{ "packageName": "com.example.app" }` |
| `DELETE` | `/api/remote/apps/system/{packageName}` | — |

**List installed apps:**
```
GET /api/remote/apps/system
Authorization: Bearer <token>
```

Response:
```json
{
  "apps": [
    { "packageName": "com.example.app", "label": "Example", "isSystem": false },
    { "packageName": "com.android.settings", "label": "Settings", "isSystem": true }
  ]
}
```

**Launch an app:**
```
POST /api/remote/apps/launch
Authorization: Bearer <token>
Content-Type: application/json

{ "packageName": "com.example.app" }
```

Response (`200 OK`):
```json
{ "status": "launched" }
```

Response (`404` if package not found):
```json
{ "error": "package not found or launch failed" }
```

**Uninstall an app:**
```
DELETE /api/remote/apps/system/com.example.app
Authorization: Bearer <token>
```

Response (`200 OK`):
```json
{ "status": "uninstall requested" }
```

**JavaScript bridge** — launch or uninstall from a web project without the Remote API:
```js
window.appcontainer.launchApp('com.example.app');
window.appcontainer.uninstallApp('com.example.app');
```

**Flutter/Dart API** — install, list, launch, and uninstall programmatically:
```dart
await MobileBridge.oemInstallApk('/sdcard/Download/app.apk');
await MobileBridge.oemLaunchApp('com.example.app');
await MobileBridge.oemUninstallApp('com.example.app');
final appsJson = await MobileBridge.oemGetInstalledApps(); // JSON array string
```

> **AVS-10 / AVS-15 touch panels only.** These endpoints and APIs are not available in the general App Container app.
> APK installation uses the Android `PackageInstaller` API (silent for system apps,
> falls back to the system install dialog otherwise). Uninstall triggers the
> system confirmation dialog.

---

## WebRTC / WHEP Proxy

App Container can bridge RTSP camera streams to WebRTC, making them viewable
in any modern browser — no plugin or native player required. The Go backend
acts as a WHEP (WebRTC HTTP Egress Protocol) server.

**No transcoding**: RTP packets are forwarded with SSRC/PT/SN rewrite.
H.264 is supported; H.265 is not yet available via WebRTC.

### Endpoints

| Method | URL | Purpose |
|--------|-----|---------|
| `POST` | `/api/stream/whep/{stream-id}` | Start a WHEP session |
| `DELETE` | `/api/stream/whep/{stream-id}` | Stop a session |
| `GET` | `/api/stream/sessions` | List active sessions (max 8) |

### Starting a session

Send a WebRTC offer SDP along with the RTSP URL:

```
POST /api/stream/whep/cam1
Content-Type: application/json

{
  "url": "rtsp://192.168.1.100:554/stream",
  "sdp": "v=0\r\no=- ..."
}
```

Response (`200 OK`):
```json
{
  "sdp": "v=0\r\no=- ..."
}
```

Apply the answer SDP to the `RTCPeerConnection` as a remote description.

### Browser example

```js
const pc = new RTCPeerConnection();
pc.addTransceiver('video', { direction: 'recvonly' });
pc.addTransceiver('audio', { direction: 'recvonly' });

pc.ontrack = e => {
  videoElement.srcObject = e.streams[0];
};

const offer = await pc.createOffer();
await pc.setLocalDescription(offer);

const resp = await fetch('http://{device}:8080/api/stream/whep/cam1', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ url: 'rtsp://...', sdp: offer.sdp })
});

const { sdp } = await resp.json();
await pc.setRemoteDescription({ type: 'answer', sdp });
```

### ICE

WHEP sessions use **localhost ICE only** — the Go server and the browser
run on the same device. No STUN/TURN required.

---

## Download

| Platform | Link |
|---|---|
| iOS (iPhone & iPad) | [App Store](https://apps.apple.com/us/app/avstudio-app-container/id6757149515) |
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
