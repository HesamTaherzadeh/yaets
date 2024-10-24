cmake_minimum_required(VERSION 3.8)
project(depth_inference)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)
find_package(CUDA REQUIRED)
find_library(CUDNN_LIBRARY cudnn HINTS /usr/lib/x86_64-linux-gnu /usr/local/cuda/lib64)
find_package(tf2_geometry_msgs REQUIRED)
find_package(PCL REQUIRED)
find_package(yaets REQUIRED)

# Manually find onnxruntime
find_package(onnxruntime REQUIRED)
# Specify C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Include directories
include_directories(include ${OpenCV_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS})

# Add executable
add_executable(slam_node src/depth_node.cpp src/model_reader.cpp)

target_link_libraries(slam_node ${onnxruntime_LIBRARY} ${OpenCV_LIBS} ${CUDA_LIBRARIES} ${CUDNN_LIBRARY} ${PCL_LIBRARIES})

ament_target_dependencies(slam_node
  yaets
  tf2_ros
  rclcpp
  pcl_conversions
  cv_bridge
  nav_msgs
  std_msgs
  sensor_msgs
  tf2_geometry_msgs
  OpenCV
  CUDA
  PCL
)

add_executable(utils_node src/utils.cpp)

ament_target_dependencies(utils_node
  rclcpp
  sensor_msgs

)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

# Install target
install(TARGETS slam_node
  DESTINATION lib/${PROJECT_NAME}
)

# Install target
install(TARGETS utils_node
  DESTINATION lib/${PROJECT_NAME}
)


install(DIRECTORY launch/
  DESTINATION share/${PROJECT_NAME}/launch
)

install(DIRECTORY urdf/
  DESTINATION share/${PROJECT_NAME}/urdf
)

install(DIRECTORY cfg/
  DESTINATION share/${PROJECT_NAME}/cfg
)

ament_package()
