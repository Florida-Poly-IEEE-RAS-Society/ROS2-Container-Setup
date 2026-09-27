# ROS 2 Jazzy Workspace

A container setup for ROS 2 Jazzy development. The image builds on `ros:jazzy-ros-base` and adds colcon, rosdep, git, vim, and tmux.

## Files

| File | Purpose |
|------|---------|
| `Containerfile` | Builds the ROS 2 Jazzy image. |
| `.github/workflows/container-publish.yml` | Builds the image and publishes it to `ghcr.io`. |
| `container-compose.yml` | Runs the image with host networking and a mounted source directory. |
| `ros_ws/src/` | ROS 2 package sources. Mounted at `/workspace/src` in the container. |

## Requirements

You need one of these tools:

- Podman with `podman compose`.
- Docker with `docker compose`.
- Apple `container` with `container-compose` (macOS on Apple silicon).

## Pull and run

A GitHub Actions workflow builds the image and publishes it to the GitHub Container Registry. The image supports `linux/amd64` and `linux/arm64`. You do not need to build the image yourself.

1. Pull the image:

   ```sh
   podman pull ghcr.io/florida-poly-ieee-ras-society/ros2-container-setup:latest
   ```

2. Start the container in the background:

   ```sh
   podman compose -f container-compose.yml up -d
   ```

3. Open a shell in the container:

   ```sh
   podman exec -it ros2_workspace bash
   ```

To use Docker, replace `podman` with `docker` in each command.

To use Apple `container`, use these commands:

```sh
container image pull ghcr.io/florida-poly-ieee-ras-society/ros2-container-setup:latest
container-compose -f container-compose.yml up -d
container exec -it ros2_workspace bash
```

To get a newer image, run the pull command again. Then run `up -d` again.

## Build the image locally

Build the image yourself only if you change the `Containerfile`:

```sh
podman compose -f container-compose.yml build
```

With Apple `container`, use `container-compose -f container-compose.yml build`.

## Image tags

The workflow in `.github/workflows/container-publish.yml` publishes these tags:

| Tag | Source |
|-----|--------|
| `latest` | The newest commit on `main`. |
| `sha-<commit>` | One specific commit. |
| `1.2.3`, `1.2` | A Git tag such as `v1.2.3`. |

Pull requests build the image to test it. The workflow does not publish pull request images.

## Inside the container

The shell sources `/opt/ros/jazzy/setup.bash` at start. The working directory is `/workspace`. Your local `./ros_ws/src` directory appears at `/workspace/src`.

Build your packages:

```sh
cd /workspace
rosdep update
rosdep install --from-paths src --ignore-src -y
colcon build
source install/setup.bash
```

## Container settings

- **`network_mode: host`**: The container shares the host network. ROS 2 nodes in the container can discover nodes on the host and the local network.
- **`ipc: host`**: The container shares host IPC. DDS shared memory transport needs this setting.
- **`:Z` volume flag**: Podman relabels the mounted directory for SELinux. Docker on systems without SELinux ignores the flag.

NOTE: On macOS, containers run inside a Linux VM. `network_mode: host` connects to the VM network, not the Mac network. ROS 2 discovery between the container and the Mac does not work in this mode.

NOTE: Apple `container` ignores `network_mode: host`. The container gets its own address on the `192.168.64.0/24` network.

- **`command: ["sleep", "infinity"]`**: Keeps the container running so you can open a shell with `exec`. Apple `container` cannot start a detached container with both `stdin_open` and `tty` set, so the file does not use those keys.

## Stop the container

```sh
podman compose -f container-compose.yml down
```
