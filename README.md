# Scan-N Plan Implementation for Spray Mold Wash (RAMP - SFSA)

# RAMP 2025 - Mold Wash Spray with Scan-n-Plan

<a href="https://docs.ros.org/en/humble/index.html"><img src="https://img.shields.io/badge/ROS 2-Humble-blue"/></a>  <a href="https://www.ros.org/reps/rep-2004.html"><img src="https://img.shields.io/badge/REP_2004-Quality_4-red"/></a>

**Project Title:**  Spray Mold Wash<br>
**Period of Performance:**  June to August 2025



## :one: Overview

**What is Mold Wash?**<br>
The application of a refractory coating onto the surface of a sand mold. Coating is typically applied in a liquid state and fully dried before use in foundry. The refractory coating _protects the sand_ from liquid steel during the casting process.

**What is a refractory coating?**<br>
The liquid is refractory particles suspended in a carrier fluid such as water, solvent, alcohol, etc. A refractory coating reduces the risk of 'burn on/in' sand to the final part surface and overall improves surface smoothness. Note, cast aluminum does NOT use mold wash due to Aluminum's lower melting point.

**Code Modularity**<br>
This project is built upon the ROS Industrial "Scan-N-Plan" workshop framework [linked here](https://github.com/ros-industrial-consortium/scan_n_plan_workshop). Dependencies inclusions have been documented in the source code.



## :two: Documentation

Project documentation includes design plans, build instructions, and usage guides. Project documentation may be found in the [/docs](docs) subfolder. Some package specific documentation is available for some packages in their own sub /docs folders.



## :two: About the Team

This project has received funding from the [Steel Founders’ Society of America (SFSA)](https://www.sfsa.org/).

| The Ohio State University | Southwest Research Institute | Iowa State University | University of Northern Iowa |
| :-: | :-: | :-: | :-: |
| <img src="https://user-images.githubusercontent.com/44533530/219718087-03c9fc38-9473-48ae-8918-024a9d2591cc.png" height="80" />  | <img src="https://user-images.githubusercontent.com/44533530/219718687-af73205d-176b-40ff-ae79-db428227fe19.png" height="80" /> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/15/Iowa_State_University_wordmark.svg/1280px-Iowa_State_University_wordmark.svg.png"  height="80" /> | <img src="https://ur.uni.edu/sites/default/files/inline-uploads/primarylogo_0.png"  height="80" />

## Build Instructions
1. Clone the application-specific ROS2 dependencies into the workspace
    ```commandLine
    cd <smw_workspace>
    vcs import src < src/sfsa-ramp-2025/dependencies.repos

    vcs import --shallow --debug src < src/scan_n_plan_workshop/dependencies_tesseract.repos
    vcs import --shallow --debug src < src/scan_n_plan_workshop/dependencies.repos

    source install/setup.bash
    rosdep install --from-paths src --ignore-src -r -y
    ```
1. Build
    ```commandLine
    colcon build --cmake-args -DTESSERACT_BUILD_FCL=OFF -DBUILD_RENDERING=OFF
    ```
1. Run the application
    ```commandLine
    cd <smw_workspace>
    source install/setup.bash
    ros2 launch ur10e_support start.launch.xml sim_robot:=<true|false> sim_vision:=<true|false>
    ```
