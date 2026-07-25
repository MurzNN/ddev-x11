# ddev-gui

DDEV add-on to enable launching GUI apps from the DDEV container using connection to the X11 server on the host.

## Installation

```sh
ddev add-on get MurzNN/ddev-x11
```

## Overview

If you want to launch a GUI program from the DDEV container, it usually fails with an error:
```
Error: Can't open display: 
```

This add-on fixes the issue by mounting the X11 Unix socket to the container and providing a `DISPLAY` env variable.

This should work well on Linux with Wayland environments too.

Probably, this way will work on MacOS and Windows too, but not tested, so share your feedback.

## Testing

```sh
ddev ssh
sudo apt update && sudo apt install x11-apps -y
xeyes
```

## Why?

Probably you have a question why do we need to run GUI apps from the container, instead of the host system, right?

Usually, it is needed to run Playwright tests using the container's environmnet and internal network with seeing the
real browser GUI window (Chromium, Firefox, Chrome, Edge) launched from the DDEV contaner with all real clicks.

This approach works with VS Code's Playwright extension run from the VS Code instance attached to the DDEV web contaier
out of the box without any special changes. 

## Details

1. Mounts the X11 Unix socket
```yaml
services:
  web:
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix:rw
```
2. Sets the DISPLAY environment variable to connect to the local X11 server.
```yaml
web_environment:
  - DISPLAY=:0
```

### Alternative solution

There is another solution to achive the same result without mounting the Unix socket:

1. Enable connections from Docker containers to the host's X11 server like this (execute on the host machine):

```sh
xhost +local:docker
```

2. Set the environment varible like this:
```sh
web_environment:
  - DISPLAY=host.docker.internal:0
```

3. Check the connection from the container to the X11 server on the host:
```sh
ddev ssh
nc -zv host.docker.internal 6000
```

This approach looks more simple, but it might not work if a firewall is enabled and blocks some network connections.

Therefore, a more stable approach is chosen by default.

