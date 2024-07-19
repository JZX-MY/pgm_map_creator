# pgm_map_creator
Create pgm map from Gazebo world file for ROS localization. Tested on Ubuntu 20.04, ROS Noetic, Boost 1.71，Gazebo 11

## Environment
Ubuntu 20.04, ROS Noetic, Boost 1.71，Gazebo 11

## Installation
### Install the dependencies.
```bash
sudo apt-get install libignition-math2-dev protobuf-compiler
```

### Clone the package to the `src` folder.
```bash
cd catkin_ws/src/
git clone https://github.com/JZX-MY/pgm_map_creator
```

### Build the catkin workspace.
```bash
cd catkin_ws
catkin_make
```

## Usage
### Copy the world file to the `worlds` folder.
```bash
cd catkin_ws/src/pgm_map_creator
mkdir worlds
cp my_world.world ~/catkin_ws/src/pgm_map_creator/worlds/
```

### Add the plugin to the world file.
Add the following code to the `my_world.world`, just before the `</world>` tag.
```xml
<plugin filename="libcollision_map_creator.so" name="collision_map_creator"/>
```

### `roslaunch` the node `gazebo_ros` to spwan the world file in the Gazebo.
```bash
cd catkin_ws/src/pgm_map_creator/launch
touch my_world.launch
```
`my_world.launch`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<launch>
  <!-- World File -->
  <arg name="world_file" default="$(find pgm_map_creator)/worlds/my_world.world"/>

  <!-- Launch Gazebo World -->
  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <arg name="use_sim_time" value="true"/>
    <arg name="debug" value="false"/>
    <arg name="gui" value="true" />
    <arg name="world_name" value="$(arg world_file)"/>
  </include>
</launch>
```
```bash
roslaunch pgm_map_creator my_world.launch
```

### `roslaunch` the node `request_publisher` to create the pgm map of the world file.
```bash
roslaunch pgm_map_creator request_publisher.launch
```

Wait for the plugin th generate map. Track the progess in the terminal that spawns the world file. Once it is done, the pgm map would be in the `map` folder of the `pgm_map_creator`.

### Create the `yaml` file for the pgm map.
You may also need a `yaml` file which provides the metadata of the map.
```
cd catkin_ws/src/pgm_map_creator/maps
touch map.yaml
```
`map.yaml`
```
image: map.pgm
resolution: 0.01
origin: [-15.0, -15.0, 0.0]
occupied_thresh: 0.65
free_thresh: 0.196
negate: 0
```

The parameters `resolution` and `origin` are based on the arguments in `request_publisher.launch`.
 
## Issues
### 1. `error: ‘int_p_NULL’ was not declared in this scope`
Add the following code to the top of  `/usr/include/boost/gil/extension/io/png_io_private.hpp`
```cpp
#define png_infopp_NULL (png_infopp)NULL
#define int_p_NULL (int*)NULL
```

### 2. `[libprotobuf ERROR google/protobuf/descriptor_database.cc:57] File already exists in database: vector2d.proto`
Change `CmakeLists` in `pgm_map_creator/msgs/CMakeLists.txt` to the following
```cmake
find_package(Protobuf REQUIRED)

set(PROTOBUF_IMPORT_DIRS)
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(PROTOBUF_IMPORT_DIRS "${ITR}/gazebo/msgs/proto")
  endif()
endforeach()

set (msgs
  collision_map_request.proto
  #${PROTOBUF_IMPORT_DIRS}/vector2d.proto
  #${PROTOBUF_IMPORT_DIRS}/header.proto
  #${PROTOBUF_IMPORT_DIRS}/time.proto
)

PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS ${msgs})
add_library(collision_map_creator_msgs SHARED ${PROTO_SRCS})
target_include_directories(collision_map_creator_msgs PUBLIC ${PROTOBUF_INCLUDE_DIRS})
target_link_libraries(collision_map_creator_msgs ${PROTOBUF_LIBRARY})
```

### 3. Cropped Map
Edit the arguments in the `launch/request_publisher.launch` for a larger map size.
```xml
<arg name="xmin" default="-15" />
<arg name="xmax" default="15" />
<arg name="ymin" default="-15" />
<arg name="ymax" default="15" />
<arg name="scan_height" default="5" />
<arg name="resolution" default="0.01" />
```

## Reference
1. Udacity: Robotic Software Engineer Nanodegree Program
2. https://github.com/udacity/pgm_map_creator
3. https://github.com/hyfan1116/pgm_map_creator
