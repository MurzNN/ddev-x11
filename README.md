[![tests](https://github.com/MurzNN/ddev-x11/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/MurzNN/ddev-x11/actions/workflows/tests.yml?query=branch%3Amain)
[![last commit](https://img.shields.io/github/last-commit/MurzNN/ddev-x11)](https://github.com/MurzNN/ddev-x11/commits)

# DDEV X11

## Overview

DDEV add-on to enable launching GUI apps from the DDEV `web` container using the X11 server on the host.

Without this add-on, GUI programs in the container usually fail with:

```
Error: Can't open display:
```

This add-on fixes that by mounting the X11 Unix socket, running `xhost +local:docker` on the host, and setting the
required environment variables in the container.

## Installation

```bash
ddev add-on get MurzNN/ddev-x11
```

## What it does

- Runs `xhost +local:docker` on the host before each `ddev start`.
- Bind-mounts `/tmp/.X11-unix` into the `web` container.
- Sets `DISPLAY=:0` in the container.
- Sets `LIBGL_ALWAYS_SOFTWARE=1` so Chromium/Mesa do not hang trying to use GPU/DRM inside Docker when no window appears.

This should work well on Linux with Wayland environments too. macOS and Windows may work but are less tested — feedback
is welcome.

## Manual testing

```bash
ddev ssh
sudo apt update && sudo apt install x11-apps -y
xeyes
```

## Why?

Running GUI apps from the container is useful when you need the container's environment and internal network while still
seeing a real browser window (Chromium, Firefox, Chrome, Edge) with real clicks — for example Playwright headed mode.

This approach works with VS Code's Playwright extension attached to the DDEV `web` container without special changes.

## macOS / rootless Docker alternative

When bind-mounting `/tmp/.X11-unix` is not possible (for example rootless Docker on macOS), use the commented
alternative in `.ddev/config.x11.yaml`:

```yaml
web_environment:
  - DISPLAY=host.docker.internal:0
```

You still need `xhost +local:docker` (or equivalent) on the host so Docker can connect to the X server.

To verify connectivity from the container:

```bash
ddev ssh
nc -zv host.docker.internal 6000
```

This network-based approach is simpler but may fail if a firewall blocks connections, so the Unix socket mount is the
default.

## Credits

**Contributed and maintained by [@MurzNN](https://github.com/MurzNN)**
