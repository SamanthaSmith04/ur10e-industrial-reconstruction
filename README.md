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
python3 -m colcon build
```

```bash
source install/setup.bash
```

## Running Support Package
```bash
ros2 launch ur10e_support start.launch.xml
```

### Running Industrial Calibration

There are 3 parts to calibration:
- **Data Collection**: Collecting image and robot position data to use for calibration
- **Intrinsic Calibration**: The properties of the camera and how these map to the 2d image (ex. focal length). The Orbbec cameras actually publish these values, which makes the process very easy!
- **Extrinsic Calibration**: Used to determine the exact location of the camera, relative to the robot and how the 2D points from the image map to a 3D position in space relative to the `world` frame.

#### Data Collection
This should only need to be done for the first time the camera is used / when the environment changes.
`ros2 launch industrial_calibration_ros data_collection.launch.xml config_file:=~/ros2_ws/src/ur10e-industrial-reconstruction/config/target_detector_config.yaml`

This `target_detector_config.yaml` file has the parameters for the 9x12 Charuco board at AIMS, but can be updated to match other boards as needed.

For calibration, you will be collecting image data of the charuco board. The board will remain stationary on the table, it is very important that the board does not move while you are collecting this data (as well as the robot cart if it is not bolted to the ground). 

The buttons in the top right of the GUI are used for data collection, and it displays how many images have been taken so far. It is reccomended to have 15-20 good images of the charuco board.

What are good images?
- Drastically different views of the board
- Majority of markers are visible to the camera (the blue dots in the image view `/annotated_image`)
- Lighting conditions are similar to useage environment

These images are used in addition to the position data of end of the robot `tool0` at the time of each image to locate the aruco marker positions in space relative to the `world` frame from the camera data (2D color feed and depth map) to perform the Extrinsic calibration.

#### Intrinsic Calibration
Since the Orbbec camera publishes the intrinsic info, it can easily be pulled from the ros topic `/camera/color/camera_info`. This will list a few parameters for the camera, we will be looking at `k`. More information on what this parameter is encoding can be found [here](https://ksimek.github.io/2013/08/13/intrinsic/). The information is laid out as an array, but `k` represents a 3x3 matrix:

$$ k =
\begin{bmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1
\end{bmatrix}
$$


#### Extrinsic Calibration

### Running Industrial Reconstruction
[Industrial Reconstruction Repo](https://github.com/ros-industrial/industrial_reconstruction)
