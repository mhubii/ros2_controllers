cmake_minimum_required(VERSION 3.5)
project(imu_sensor_broadcaster)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(THIS_PACKAGE_INCLUDE_DEPENDS
    controller_interface
    generate_parameter_library
    hardware_interface
    pluginlib
    rclcpp
    rclcpp_lifecycle
    realtime_tools
    sensor_msgs
)

find_package(ament_cmake REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()

generate_parameter_library(imu_sensor_broadcaster_parameters
  src/imu_sensor_broadcaster_parameters.yaml
)

add_library(
  imu_sensor_broadcaster SHARED
  src/imu_sensor_broadcaster.cpp
)
target_include_directories(imu_sensor_broadcaster
  PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE $<INSTALL_INTERFACE:include>
)
ament_target_dependencies(imu_sensor_broadcaster ${THIS_PACKAGE_INCLUDE_DEPENDS})
# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(imu_sensor_broadcaster PRIVATE "IMU_SENSOR_BROADCASTER_BUILDING_DLL")
target_link_libraries(imu_sensor_broadcaster
  imu_sensor_broadcaster_parameters
)

pluginlib_export_plugin_description_file(
  controller_interface imu_sensor_broadcaster.xml)

if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(controller_manager REQUIRED)
  find_package(hardware_interface REQUIRED)
  find_package(ros2_control_test_assets REQUIRED)

  ament_add_gmock(
    test_load_imu_sensor_broadcaster
    test/test_load_imu_sensor_broadcaster.cpp
  )
  target_include_directories(test_load_imu_sensor_broadcaster PRIVATE include)
  ament_target_dependencies(test_load_imu_sensor_broadcaster
    controller_manager
    hardware_interface
    ros2_control_test_assets
  )

  ament_add_gmock(
    test_imu_sensor_broadcaster
    test/test_imu_sensor_broadcaster.cpp
  )
  target_include_directories(test_imu_sensor_broadcaster PRIVATE include)
  target_link_libraries(test_imu_sensor_broadcaster
    imu_sensor_broadcaster
  )
  ament_target_dependencies(test_imu_sensor_broadcaster
    hardware_interface
  )
  ament_target_dependencies(test_load_imu_sensor_broadcaster
    controller_manager
    hardware_interface
    ros2_control_test_assets
  )
endif()

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS
    imu_sensor_broadcaster
    imu_sensor_broadcaster_parameters
  EXPORT export_imu_sensor_broadcaster
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  INCLUDES DESTINATION include
)

ament_export_targets(export_imu_sensor_broadcaster HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()
