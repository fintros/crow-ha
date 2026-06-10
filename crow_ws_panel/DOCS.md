# Elecrow ESP32P4 - Home Assistant Browser Panel

Runs a headless Chromium kiosk inside the add-on, captures its screen, and
streams changed regions as JPEG tiles over a binary WebSocket.

The add-on is built `FROM grovety/crow-ws-server` (set in `build.yaml`), so the
heavy bits are pulled prebuilt from Docker Hub.
