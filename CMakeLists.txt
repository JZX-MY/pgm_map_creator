cmake_minimum_required(VERSION 2.8.8)
project(pgm_map_creator)

set (CMAKE_CXX_FLAGS "-g -Wall -std=c++17")

include (FindPkgConfig)
if (PKG_CONFIG_FOUND)
  pkg_check_modules(GAZEBO gazebo)
  pkg_check_modules(SDF sdformat)
endif()

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
)

find_package(Boost REQUIRED COMPONENTS system)
find_package(Threads REQUIRED)


catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES pgm_map_creator
#  CATKIN_DEPENDS roscpp rospy
#  DEPENDS system_lib
)


include_directories(
  ${catkin_INCLUDE_DIRS}

  ${GAZEBO_INCLUDE_DIRS}
  ${SDF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}/msgs
)

link_directories(${GAZEBO_LIBRARY_DIRS} ${CMAKE_CURRENT_BINARY_DIR}/msgs)
add_subdirectory(msgs)

add_executable(request_publisher src/request_publisher.cc)
target_link_libraries(request_publisher collision_map_creator_msgs ${Boost_LIBRARIES} ${GAZEBO_LIBRARIES} ${SDF_LIBRARIES} Threads::Threads)  # add Threads::Threads
add_dependencies(request_publisher collision_map_creator_msgs)

add_library(collision_map_creator SHARED src/collision_map_creator.cc)
target_link_libraries(collision_map_creator collision_map_creator_msgs ${Boost_LIBRARIES} ${GAZEBO_LIBRARIES} ${SDF_LIBRARIES} Threads::Threads)  # add Threads::Threads
add_dependencies(collision_map_creator collision_map_creator_msgs)


