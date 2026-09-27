# ROS 2 Jazzy Workspace

A container setup for ROS 2 Jazzy development. The image builds on `ros:jazzy-ros-base` and adds colcon, rosdep, git, vim, and tmux.

## Files

| File | Purpose |
|------|---------|
| `Containerfile` | Builds the ROS 2 Jazzy image. |
| `container-compose.yml` | Runs the image with host networking and a mounted source directory. |
| `ros_ws/src/` | ROS 2 package sources. Mounted at `/workspace/src` in the container. |

## Requirements

- Podman with `podman compose`, or Docker with `docker compose`.

## Build and run

1. Build the image:

   ```sh
   podman compose -f container-compose.yml build
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

## Stop the container

```sh
podman compose -f container-compose.yml down
```
