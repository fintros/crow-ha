# Crow HA Add-on

Home Assistant add-on repository for the **Crow WS Browser Panel**.

The add-on is a thin layer over the prebuilt Docker Hub image
[`grovety/crow-ws-server`](https://hub.docker.com/r/grovety/crow-ws-server),
built from the [crow-ws-server](https://github.com/grovety/crow-ws-server)
source repo. Supervisor builds `crow_ws_panel/Dockerfile`
(`FROM grovety/crow-ws-server:<version>`) on install — no compilation on the
device.

## Install

Settings → Add-ons → Add-on Store → ⋮ → **Repositories** → add:

```
https://github.com/fintros/crow-ha
```

Then install **Crow WS Browser Panel**. See `crow_ws_panel/DOCS.md` for options.

## Bumping the version

1. Publish a new base image: `./build-and-push.ps1 -Version X.Y.Z` in the
   crow-ws-server repo.
2. Update `crow_ws_panel/build.yaml` `build_from` tag and `crow_ws_panel/config.yaml`
   `version` to `X.Y.Z`.
