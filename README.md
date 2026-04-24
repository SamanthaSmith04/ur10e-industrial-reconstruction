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

**Note: replace `<PATH_TO_YOUR_WORKSPACE>` with your workspace path**
```bash
ros2 launch industrial_calibration_ros data_collection.launch.xml config_file:=<PATH_TO_YOUR_WS>/src/ur10e-industrial-reconstruction/config/target_detector_config.yaml`
```

This will launch RViz and provides displays for both data collection and extrinsic calibration!

The `target_detector_config.yaml` file has the parameters for the 9x12 Charuco board at AIMS, but can be updated to match other boards as needed.

For calibration, you will be collecting image data of the charuco board. The board will remain stationary on the table, it is very important that the board does not move while you are collecting this data (as well as the robot cart if it is not bolted to the ground). 

The buttons in the top right of the GUI are used for data collection, and it displays how many images have been taken so far. It is reccomended to have 15-20 good images of the charuco board.

What are good images?
- Drastically different views of the board
- Majority of markers are visible to the camera (the blue dots in the image view `/annotated_image`)
- Lighting conditions are similar to useage environment

These images are used in addition to the position data of end of the robot `tool0` at the time of each image to locate the aruco marker positions in space relative to the `world` frame from the camera data (2D color feed and depth map) to perform the Extrinsic calibration.

The data will be saved to the `/tmp/calibration` folder, it is important that you move this to another location if you wish to keep this data, since the `tmp` folder is temporary and will periodically be cleared.

#### Intrinsic Calibration
Since the Orbbec camera publishes the intrinsic info, it can easily be pulled from the ros topic `/camera/color/camera_info`. This will list a few parameters for the camera, we will be looking at `k`. More information on what this parameter is encoding can be found [here](https://ksimek.github.io/2013/08/13/intrinsic/). The information is laid out as an array, but `k` represents a 3x3 matrix:

$$ k =
\begin{bmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1
\end{bmatrix}
$$

This matrix is flattened for the ROS message to make it easier to write out:

$$ k =
\begin{bmatrix}
f_x \\
0 \\
c_x \\
0 \\
f_y \\
c_y \\
0 \\
0 \\
1
\end{bmatrix}
$$

These values are used for the intrinsic calibration, and can be added to the camera info tab in the GUI.


#### Extrinsic Calibration
Extrinsic calibration is used to locate the exact position of the camera lens relative to the robot. Combining this position with depth data is what allows us to convert a 2D location on an image into a 3D coordinate relative to the `world` frame.

Extrinsic calibration can either be launched using:

```bash
ros2 run industrial_calibration industrial_calibration_extrinsic_hand_eye_calibration_app
```

OR, if the data collection launch is already running from before, it is also included in that.

##### Extrinsic Calibration Steps:
1. Load observations

- The observation file is in the folder we saved earlier in data collection. This file is a `.yaml` file that maps each image to the stored pose at the time it was taken.
- The loaded images will likely say no markers detected, this is fine. Depending on the board used, the markers may not match the current target finder configuration, which is set in the next step.
   
2. Update target finder configuration
- These values should be the same as before for data collection, unfortunately this data is not auto-populated from the loaded `yaml` file and must be manually added.
- Clicking on the loaded observation images should now resolve the error from before and the markers should be labelled in the image view.

3. Update camera intrinsics
- Using the values from `ros2 topic echo /camera/color/camera_info` in the K parameter as discussed before, update the `fx`, `fy`, `cx`, and `cy` parameters (note: these are not in the same order in the GUI as they appear in the topic).

4. Update estimated camera mount position
- All of the observation poses are captured from the `tool0` position on the robot, which is the end of the robot arm. This estimate is used to provide an initial guess to the program to calculate where the camera lens is based on the image and pose data.
- Measure the position: `xyz` and orientation: `xyzw` of the camera lens, relative to the robot flange. Unless the camera is rotated, in most cases the orientation can be ignored and left as `0 0 0 1`.
- All units in the GUI are required to be in `mm`

5. Update estimated transform to calibration board
- The position of the calibration board is also needed to help estimate where points are in 3D from the 2D images. Each of the charuco markers has a specific ID, with a known position on the board.
- Measure to the top right corner of the board from the base of the robot (`base_link`). Since `base_link` is centered in the middle of the robot base, you will need to measure from that point.
- Determine how the board is oriented:
  - The robot base is usually oriented with the z-axis pointing up, the calibration board will have the z-axis pointing into the board.
  - [This website](https://www.andre-gaschler.com/rotationconverter/) can help with converting rotations into quanternions, but for most cases, it is reccomended to place the board so it is parallel to the robot base frame, to make the orientation easier to find.
  - The robot frame can be viewed using the `tf` display in RViz.

6. Run calibration
- Pressing the play button runs calibration. Depending on how many images were collected, this may take some time.
- If the calibration succeeds it will populate the window with data about the calibration. The most important numbers will be the costs, which is at the top of the display.
  - Two numbers are listed. The cost represents the difference between the position of the marker IDs across all of the images in the dataset:
    - Initial cost per observation: The distances between markers across all images before calibration (will usually be very high)
    - Final cost per observation: The distances between markers across all images after calibration. This number should be as low as possible, ideally < 5. The lower this number is, the more accurate the reconstruction will be.
   
7. Save calibration
- Don't forget to hit save before closing the GUI!
- This file can be saved to the `config` folder in this repo, or wherever you'd like to store your calibration data.
 

### Running Industrial Reconstruction
[Industrial Reconstruction Repo](https://github.com/ros-industrial/industrial_reconstruction)

```bash
ros2 service call /start_reconstruction industrial_reconstruction_msgs/srv/StartReconstruction "tracking_frame: 'camera_color_optical_frame'
relative_frame: 'world'
translation_distance: 0.0
rotational_distance: 0.0
live: true
tsdf_params:
  voxel_length: 0.005
  sdf_trunc: 0.04
  min_box_values: {x: -1.0, y: -1.0, z: -1.0}
  max_box_values: {x: 1.0, y: 3.0, z: 1.2}
rgbd_params: {depth_scale: 1000.0, depth_trunc: 0.75, convert_rgb_to_intensity: false}"
```

```bash
ros2 service call /stop_reconstruction industrial_reconstruction_msgs/srv/StopReconstruction "archive_directory: '~/Desktop'
mesh_filepath: '~/Desktop/results_mesh.ply'
normal_filters: [{ normal_direction: {x: 0.0, y: 0.0, z: 1.0}, angle: 90}]
min_num_faces: 1000"
```
