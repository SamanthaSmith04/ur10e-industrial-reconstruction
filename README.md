# UR10e support

## Workspace Setup

Follow the installation instructions found on the [Orbbec documentation](https://orbbec.github.io/OrbbecSDK_ROS2/en/source/camera_devices/2_installation/build_the_package.html). This can either be done through downloading binaries or building from source.

```bash
mkdir -p ~/ros2_ws/src
```

```bash
cd ~/ros2_ws/src
```

Clone this repo into the `src` folder
```bash
cd ~/ros2_ws
```

```bash
vcs import src < src/ur10e-industrial-reconstruction/dependencies.repos
```

```bash
colcon build
```

```bash
source install/setup.bash
```

## Running Support Package
```bash
ros2 launch ur10_support start.launch.xml
```
