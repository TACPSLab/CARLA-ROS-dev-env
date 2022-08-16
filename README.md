## Cross-platform (untested)

Some of the configurations in the Compose file are only applicable to Linux. Specifically,
```yml
# X11 forwarding to view GUI applications running on containers
environment:
  DISPLAY: ${DISPLAY}
volumes:
  - /tmp/.X11-unix:/tmp/.X11-unix:rw

[...]

network_mode: host  # access host port from container
```
The Compose file can be adapted for use on Windows and macOS by modifying those configurations.

Please also note that
```yml
devices:
- driver: nvidia
  device_ids: ['0']
```
should be set to requisitioned GPU(s).

## How to use

Clone this repo with `--recurse-submodules` option.

Set `CARLA_VERSION` and `ROS_DISTRO` in the `.env` file to the demanded version/distro. Checkout the carla-ros-bridge (ros-bridge for short) submodule to the tag/branch same as the value of `CARLA_VERSION`; e.g. set `CARLA_VERSION=0.9.11` and `cd /path/to/ros-bridge && git checkout 0.9.11`.

Executing `docker compose build` in the directory contains `docker-compose.yml` pulls the image of CARLA and builds the image for ros-bridge.

Start CARLA and ros-bridge simultaneously: `docker compose up -d`

Follow the logs of two containers: `docker compose log -f`

Note that the initial ros-bridge connection may fail because the timeout is [set too short](ros-bridge/carla_ros_bridge/src/carla_ros_bridge/bridge.py#392) and attempt to reconnect. Please verify that ros-bridge is successfully connecting to CARLA by monitoring the log.

### Workspace

The first half explains how to launch the containerised CARLA and ros-bridge using this repo. The second half will discuss how to use this environment for development.

It is possible to directly develop on the host which has corresponding ROS distro (that ros-bridge container is using) installed. However, the preferred approach is to start a containerised workspace features corresponding ROS distro and develop inside the container. For example, CARLA 0.9.11 and ros-bridge 0.9.11 built upon ROS **Noetic** have been started on a **Linux** host which has multiple **Nvidia** GPUs. Executing
```
docker run -d -t --name workspace --net=host --privileged --gpus 0 -e NVIDIA_VISIBLE_DEVICES=all -e NVIDIA_DRIVER_CAPABILITIES=all -e QT_X11_NO_MITSHM=1 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw osrf/ros:noetic-desktop-full
```
starts a container, which can display GUI on the host and has access to host's first GPU, from `osrf/ros:noetic-desktop-full` image. Next, attach the container using VSCode with `Remote-Containers` plugin installed. Then, open a directory inside the container as project root using the intergrated file explorer within VSCode. Thus far, a basic containerised workspace has been built.
