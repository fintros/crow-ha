# Home Assistant Browser for Elecrow ESP32P4 Advance panel

Show a Home Assistant dashboard on remote embedded touch screens.

Unlike a local kiosk, this add-on does **not** drive a monitor attached to your
Home Assistant server. It runs a headless Chromium inside a virtual X display,
captures the screen, JPEG-compresses only the changed regions, and streams them
over a binary WebSocket (container port `3000`) to one (in the current version) Elecrow ESP32P4 Advance panel. The
panel draws the tiles and sends touch and navigation events back over the same
socket.

- **One panel per add-on instance.** The server is single-client: a new
  connection drops the previous panel and restarts the capturer. (_It is the limitation of the current version only_)
- **The panel supplies its own settings** — resolution, JPEG quality, and Home
  Assistant credentials — in the WebSocket query string. See *Connecting a panel*.
- **Login is remembered per panel.** After the first successful login the Home
  Assistant session is saved and reused, so the panel skips the login screen on
  later connections. See *Session persistence*.

**NOTE:** This add-on has no display on the server itself. You see its output only
on the connected panel device.

**NOTE:** Credentials travel in the WebSocket query string and are not encrypted.
Keep the panel and Home Assistant on a trusted network. (_It is the limitation of the current version only_)

______________________________________________________________________

## Installation

1. Add this repository to your add-on store:

   [![Open your Home Assistant instance and show the add Add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Ffintros%2Fcrow-ha)

   Or manually: **Settings → Add-ons → Add-on Store → ⋮ → Repositories**, paste
   `https://github.com/fintros/crow-ha`, then **Add → Close**.

2. Open **Elecrow ESP32P4 interface**, press **Install**, and wait for the build to
   finish. Supervisor pulls the prebuilt `grovety/crow-ws-server` base image, so
   there is no compilation on the device.

3. (Optional) Open the **Network** panel and pick the host port — see
   *Configuration*.

4. Press **Start**.

5. Point your panel device at the add-on — see *Connecting a panel*.

______________________________________________________________________

## Configuration

For now the **server port is the only adjustable parameter.** Everything else is
baked into the image or supplied per-connection by the panel, so the
**Configuration** tab is intentionally empty.

### Server port

The WebSocket server always listens on container port **3000**. To change the
port reachable on your network, open the add-on's **Network** panel, map
`3000/tcp` to the host port you want, and restart the add-on. Use that port in
the panel's connection URL.

### Fixed defaults

These are set in the image and are not user-configurable yet:

| Setting | Value | Meaning |
|---------|-------|---------|
| Start URL | `http://homeassistant:8123` | Page the kiosk opens. `homeassistant` is the Supervisor's internal hostname for your HA instance. |
| Capture resolution | `1024 × 600` | Virtual display size = captured framebuffer size. |
| Frame rate | `15` fps | Capture/stream rate. |

______________________________________________________________________

## Session persistence

After the first successful login, the add-on saves the Home Assistant frontend
session (`hassTokens`) to `/data/tokens/<id>.json`, keyed by the panel's `id`.
On later connections with the same `id` it re-injects the session and skips the
username/password autologin. If the stored session is revoked or invalid, Home
Assistant redirects to the login page and the credential autologin takes over,
overwriting the file with a fresh session.

`/data` is persistent and private to the add-on, so saved sessions survive
restarts and updates.

> **Security:** the saved session contains a non-expiring `refresh_token` (full
> account access) stored in plaintext under `/data`. Keep the host trusted. To
> force a fresh login, stop the add-on and delete the panel's file in
> `/data/tokens`.

______________________________________________________________________

## How it works

- **`bridge`** (compiled C++) runs the WebSocket server, memory-maps the Xvfb
  framebuffer, diffs it into tiles, JPEG-compresses the changed tiles, and sends
  them to the panel.
- **`browser_ctl.js`** drives Chromium over the DevTools protocol: it dispatches
  the panel's touch events, performs the Home Assistant autologin, reports the
  current URL, and manages the saved session.
- The two communicate over a local Unix socket. The panel only ever speaks the
  binary WebSocket protocol.

______________________________________________________________________

## Troubleshooting

- **Panel shows nothing / connection refused.** Confirm the add-on is **Started**
  and that you are connecting to the host port mapped in the **Network** panel.
- **Stuck on the login screen.** Check that you entered correct user and password in the pannel settings.
  Check the add-on log for `[JS] Login failed`.
- **Image is cropped or scaled.** The capture is fixed at 1024×600; set the panel
  to that resolution.
- **Want a clean login.** Press logaout in the Home assistant interface or Stop the add-on and delete `/data/tokens/<id>.json`.
- **Wrong dashboard / HA not reachable.** The start URL is fixed to
  `http://homeassistant:8123`. If your HA is not reachable there, that is not yet
  configurable from the add-on.