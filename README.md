# Model Checking Environment: Storm + Spot (Docker)

A ready-to-use Docker environment for probabilistic model checking with [Storm](https://www.stormchecker.org/) / [stormpy](https://github.com/moves-rwth/stormpy) and LTL-to-automata translation with [Spot](https://spot.lre.epita.fr/), including GUI support (matplotlib / PyQt / X11) for visualizing plots and automata.

## Services

The environment is managed via **Docker Compose** ([compose.yaml](compose.yaml)) and consists of three services:

- **`spot`** — image with [Spot 2.11.6](https://spot.lre.epita.fr/) (LTL → automata), built from [spot/Dockerfile](spot/Dockerfile).
- **`storm`** — Ubuntu image with [Storm](https://github.com/moves-rwth/storm) + [stormpy](https://github.com/moves-rwth/stormpy) compiled from source, defined in [storm/Dockerfile](storm/Dockerfile). Also available: [storm/Dockerfile_stable](storm/Dockerfile_stable), which starts from the official `movesrwth/storm:stable` image instead of building from source (faster build, less control over the exact version).
- **`merged_service`** — final image that combines Storm and Spot (copies `/opt/spot-2.11.6` from the `spot_image` image into `test_ubuntu`), defined in [merged/Dockerfile](merged/Dockerfile). This is the service meant for running scripts that use both tools.

A [.devcontainer/](.devcontainer/) is included for those who prefer working from VS Code with the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Docker Compose plugin.
- For the GUI (matplotlib / PyQt / X11): a running X server on the host and access to `/tmp/.X11-unix`. On Linux hosts you may need to enable the display:
  ```sh
  xhost +local:docker
  ```

## Setup

1. Copy the example environment file:
   ```sh
   cp .env.example .env
   ```
   The `.env` file is read automatically by `docker compose` and contains the variables used by the `storm` and `merged_service` services for graphical rendering (`DISPLAY`, `QT_X11_NO_MITSHM`, `XAUTHORITY`, `LIBGL_ALWAYS_SOFTWARE`, `XDG_RUNTIME_DIR`). Adjust the values for your environment if needed.

2. If `~/.Xauthority` doesn't exist on your host, make sure the path is correct (in [compose.yaml](compose.yaml) it's already referenced as `${HOME}/.Xauthority`).

## Building the images

From the project folder:
```sh
docker compose build
```
or build a single service:
```sh
docker compose build spot
docker compose build storm
docker compose build merged_service
```
> Note: `merged_service` depends on the `spot_image` and `test_ubuntu` images, so `spot` and `storm` must be built **before** `merged_service`.

## Starting the working container

The main service for running scripts is `merged_service`. The [storm_ws/](storm_ws/) folder is mounted at `/storm_ws` inside the container, and is where you should place your PRISM models and Python scripts:
```sh
docker compose run --rm merged_service bash
```
You'll be dropped into an interactive shell in `/storm_ws`. File changes are visible from both the host and the container.

Alternatively, you can enter the individual services directly:
```sh
docker compose run --rm storm bash
docker compose run --rm spot bash
```

## Cleanup

```sh
docker compose down              # stop the services
docker image rm storm_spot_merged test_ubuntu spot_image   # remove the images if you want to rebuild
```

## License

This repository (the Docker environment) is distributed under the [MIT](LICENSE) license. Storm and Spot are third-party projects with their own licenses: refer to their respective repositories for details.
