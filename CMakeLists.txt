cmake_minimum_required(VERSION 3.5.0)
project(ur10e_support)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Extract package name and version
find_package(rclcpp REQUIRED)
find_package(ros_industrial_cmake_boilerplate REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(ur_description REQUIRED)
find_package(ur_robot_driver REQUIRED)

install(
  DIRECTORY launch
            config
            urdf
            meshes
  DESTINATION share/${PROJECT_NAME}/)


ament_export_dependencies(behaviortree_ros2 snp_msgs geometry_msgs Eigen3)
ament_package()
