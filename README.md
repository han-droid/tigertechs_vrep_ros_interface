# How to install vrep_ros_intferface

[Reference 1](https://github.com/LCAS/zoidbot/wiki/VREP-RosInterface-Setup) 
[Reference 2](https://github.com/CoppeliaRobotics/v_repExtRosInterface)

#### 1. Make catkin workspace

```
$ mkdir -p vrep_ws/src
$ cd vrep_ws/src
$ catkin init
```

#### 2. Clone github code
```
$ git clone --recursive https://github.com/CoppeliaRobotics/v_repExtRosInterface.git vrep_ros_interface
```

#### 3. Build
```
$ cd ..
$ catkin build
```



# How to add Kinova msg, srv to vrep_ros_interface

#### 1. kortex_driver/msg, kortex_driver/srv
First, ros_kortex package is needed. And need to make sure that messages, services are in ros working space, so that vrep_ros_interface package can reach ros_kortex related messages and services. 

After successfully build for ros_kortex, let's try to check ros message, ros service.

```
$ source devel/setup.bash
$ rosmsg show kortex_driver/Twist
```
Using `rosmsg` command, you can check message. But we will see some error here.  Need to do some trick to make it work correctly.

Under `ros_kortex/kortex_dirver/msg` and `ros_kortex/kortex_driver/srv` directory, there are sub-directories and under these sub-directories, we can see messages and services. **But `catkin build` system didn't recognize sub-directories.**  So put all the message files into under `msg` directory. And all the service files into under `srv` directory.

And modify `CMakeLists.txt` file like following.

```
file(GLOB_RECURSE generated_files RELATIVE ${PROJECT_SOURCE_DIR} "src/generated/*.cpp")
file(GLOB_RECURSE non_generated_files RELATIVE ${PROJECT_SOURCE_DIR} "src/non-generated/*.cpp")

## Find all auto-generated subdirectories in msg/generated
#file(GLOB children RELATIVE ${PROJECT_SOURCE_DIR}/msg/generated ${PROJECT_SOURCE_DIR}/msg/generated/*)
#set(msg_generated_dir_list "")
#foreach(child ${children})
#    if(IS_DIRECTORY ${PROJECT_SOURCE_DIR}/msg/generated/${child})
#      list(APPEND msg_generated_dir_list ${child})
#    endif()
#endforeach()

## Find all auto-generated subdirectories in srv/generated
#file(GLOB children RELATIVE ${PROJECT_SOURCE_DIR}/srv/generated ${PROJECT_SOURCE_DIR}/srv/generated/*)
#set(srv_generated_dir_list "")
#foreach(child ${children})
#    if(IS_DIRECTORY ${PROJECT_SOURCE_DIR}/srv/generated/${child})
#      list(APPEND srv_generated_dir_list ${child})
#    endif()
#endforeach()

## declare ROS messages and services
#add_message_files(DIRECTORY msg/non_generated)
#add_message_files(DIRECTORY msg/generated)
#foreach(sub_dir ${msg_generated_dir_list})
#    add_message_files(DIRECTORY msg/generated/${sub_dir})
#endforeach()

#add_service_files(DIRECTORY srv/non_generated)
#foreach(sub_dir ${srv_generated_dir_list})
#    add_service_files(DIRECTORY srv/generated/${sub_dir})
#endforeach()

add_message_files(DIRECTORY msg)    # <--- Added  ----------------------------->
add_service_files(DIRECTORY srv)    # <--- Added ------------------------------>

## generate added messages and services
generate_messages(DEPENDENCIES std_msgs actionlib_msgs)

```
Comment adding sub-directories part. And just add two lines for msg and srv.

And build again
```
$ catkin clean
$ catkin build
```

#### 2. Add msg, srv to vrep_ros_interface

Let's edit `vrep_ros_interface/meta/services.txt` and add `SendTwistCommand` service.

```
dynamic_reconfigure/Reconfigure
roscpp/Empty
roscpp/GetLoggers
roscpp/SetLoggerLevel
std_srvs/Empty
std_srvs/Trigger
kortex_driver/SendTwistCommand     # <--  this is added 
```

Look in `SendTwistCommand.srv` file, will notice that there are two messages. `TwistCommand` and `Empty`.  And in the `TwistCommand` message, there is `Twist` message too. So let's add these messages in `vrep_ros_interface/meta/message.txt`.

```
...
...
visualization_msgs/InteractiveMarkerPose
visualization_msgs/InteractiveMarkerUpdate
visualization_msgs/Marker
visualization_msgs/MarkerArray
visualization_msgs/MenuEntry
kortex_driver/Twist                  # <-- this is added
kortex_driver/TwistCommand           # <-- this is added
kortex_driver/Empty                  # <-- this is added
```

#### 3. Add `kortex_driver` package for catkin dependency

Edit `vrep_ros_interface/CMakeLists.txt` and `vrep_ros_interface/package.xml` like following.

```
cmake_minimum_required(VERSION 3.0.0)
project(vrep_ros_interface)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG")

set(PKG_DEPS
    roscpp
    rosconsole
    cv_bridge
    image_transport
    tf
    roslib
    actionlib_msgs
    control_msgs
    diagnostic_msgs
    geometry_msgs
    map_msgs
    nav_msgs
    pcl_msgs
    sensor_msgs
    shape_msgs
    std_msgs
    tf2_geometry_msgs
    tf2_msgs
    tf2_sensor_msgs
    trajectory_msgs
    visualization_msgs
    kortex_driver)                     # <--- This is added

find_package(catkin REQUIRED COMPONENTS ${PKG_DEPS})
...
...
```

```
...
...
  <depend>tf2_geometry_msgs</depend>
  <depend>tf2_msgs</depend>
  <depend>tf2_sensor_msgs</depend>
  <depend>trajectory_msgs</depend>
  <depend>visualization_msgs</depend>
  <depend>rosconsole</depend>
  <depend>roslib</depend>
  <depend>kortex_driver</depend>         # <--- This is added 
</package>
```

#### 4. Build again

Before build again, make sure that kortex_driver messages are accessible.  Let's check using `rosmsg`.

```
$ rosmsg show kortex_driver/Twist
```
If can not show correctly,  `source ros_kortex/devel/setup.bash`  again. Or check things again from the first.

Now let's build again.

```
$ catkin clean
$ catkin build
```
#### 5. Copy shared object to VREP

```
$ cp -iv devel/lib/libv_repExtRosInterface.so "$VREP_ROOT/"
```
