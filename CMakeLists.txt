cmake_minimum_required(VERSION 3.14)
project(livox_ros_driver2)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 14)

list(INSERT CMAKE_MODULE_PATH 0 "${PROJECT_SOURCE_DIR}/cmake/modules")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()

include(cmake/version.cmake)
message(STATUS "${PROJECT_NAME} version: ${LIVOX_ROS_DRIVER2_VERSION}")

add_definitions(-DBUILDING_ROS2)

# Dependencies
find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()
find_package(PCL REQUIRED)
find_package(std_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)

add_subdirectory(3rdparty/Livox-SDK2)

# APR
find_package(PkgConfig)
pkg_check_modules(APR apr-1)

# Messages
set(LIVOX_INTERFACES livox_interfaces2)

rosidl_generate_interfaces(${LIVOX_INTERFACES}
  "msg/CustomPoint.msg"
  "msg/CustomMsg.msg"
  DEPENDENCIES builtin_interfaces std_msgs
)

# PCL
add_definitions(${PCL_DEFINITIONS})

# Library
ament_auto_add_library(${PROJECT_NAME} SHARED
  src/livox_ros_driver2.cpp
  src/lddc.cpp
  src/driver_node.cpp
  src/lds.cpp
  src/lds_lidar.cpp

  src/comm/comm.cpp
  src/comm/ldq.cpp
  src/comm/semaphore.cpp
  src/comm/lidar_imu_data_queue.cpp
  src/comm/cache_index.cpp
  src/comm/pub_handler.cpp

  src/parse_cfg_file/parse_cfg_file.cpp
  src/parse_cfg_file/parse_livox_lidar_cfg.cpp

  src/call_back/lidar_common_callback.cpp
  src/call_back/livox_lidar_callback.cpp
)

add_dependencies(${PROJECT_NAME} ${LIVOX_INTERFACES}__rosidl_typesupport_cpp)

target_include_directories(${PROJECT_NAME} PUBLIC
  ${PCL_INCLUDE_DIRS}
  ${APR_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/Livox-SDK2/include
  3rdparty
  src
)

# Message linking
rosidl_get_typesupport_target(
  cpp_typesupport_target
  ${LIVOX_INTERFACES}
  "rosidl_typesupport_cpp"
)

target_link_libraries(${PROJECT_NAME}
  livox_lidar_sdk_shared
  ${cpp_typesupport_target}
  ${PCL_LIBRARIES}
  ${APR_LIBRARIES}
)

# Node
rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "livox_ros::DriverNode"
  EXECUTABLE ${PROJECT_NAME}_node
)

# Tests
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_include_directories(
  src
  ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/Livox-SDK2/include
)

ament_auto_package(
  INSTALL_TO_SHARE
  config
  launch_ROS2
)
