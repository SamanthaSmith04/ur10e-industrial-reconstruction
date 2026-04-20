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
vcs import src < src/industrial_calibration_ros/dependencies.repos
```

```bash
python3 -m venv venv --system-site-packages

source venv/bin/activate

pip install open3d
```

```bash
colcon build
```

```bash
source install/setup.bash
```

## Running Support Package
```bash
ros2 launch ur10e_support start.launch.xml
```

### Running Industrial Reconstruction
[Industrial Reconstruction Repo](https://github.com/ros-industrial/industrial_reconstruction)
