# How to setup vrep_ros_interface

## A. git clone and build
#### 1. Make catkin workspace

```
$ mkdir -p vrep_ws/src
$ cd vrep_ws/src
$ catkin init
```

#### 2. Clone vrep_ros_interface github code
```
$ git clone --recursive https://github.com/CoppeliaRobotics/v_repExtRosInterface.git vrep_ros_interface
```

#### 3. Build
```
$ cd ..
$ catkin build
```

## B. Add new msg or srv
#### 1. Clone tigertechs_vrep_ros_interface
```
$ git clone https://github.com/han-droid/tigertechs_vrep_ros_interface
```

#### 2. Replace files 
Replace following files 
> meta/messages.txt
> meta/services.txt
> CMakeLists.txt
> package.xml

#### 3. Build again
```
$ catkin clean
$ catkin build
```