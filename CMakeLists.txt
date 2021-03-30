cmake_minimum_required(VERSION 3.0.2)
project(gazebo_ros_ray_plugin)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  sensor_msgs
  tf
  gazebo_ros
)
find_package(gazebo REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")

catkin_package(
  INCLUDE_DIRS include ${GAZEBO_INCLUDE_DIRS}
  LIBRARIES gazebo_ros_ray_sensor
  CATKIN_DEPENDS roscpp sensor_msgs gazebo_ros
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
)

# link_directories(
#   ${GAZEBO_LIBRARY_DIRS}
# )

add_library(gazebo_ros_ray_sensor src/gazebo_ros_ray_sensor.cpp)
target_link_libraries(gazebo_ros_ray_sensor ${catkin_LIBRARIES})

install(TARGETS gazebo_ros_ray_sensor 
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
