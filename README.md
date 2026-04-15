# UR10e support

## Workspace Setup

Follow the installation instructions found on the [Orbbec documentation](https://orbbec.github.io/OrbbecSDK_ROS2/en/source/camera_devices/2_installation/build_the_package.html)

`mkdir -p ~/ros2_ws/src`

`cd ~/ros2_ws/src`

Clone this repo into the `src` folder

`cd ~/ros2_ws`

`vcs import src < src/ur10e-industrial-reconstruction/dependencies.repos`

`colcon build`

`source install/setup.bash`
