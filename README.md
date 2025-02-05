---

# TurtleBot3 Autonomous Driving Setup Guide

## Project Overview

This project aims to set up and calibrate a TurtleBot3 Burger robot for autonomous driving using ROS 1 Noetic. The setup includes installing necessary packages, calibrating the camera, configuring lane detection, and implementing autonomous driving capabilities. By following this guide, you will be able to make your TurtleBot3 navigate autonomously within predefined lanes using a Raspberry Pi camera module.

## Acknowledgment

I extend my sincere appreciation to the Integral University Robotics Lab ([https://www.robotics.iul.ac.in/](https://www.robotics.iul.ac.in/)) for their invaluable support throughout the development of the ARM2.0 project. Their generous provision of funds, tools, and a conducive environment for research and innovation has been instrumental in bringing this project to fruition. I am deeply grateful for their guidance and expertise, which have played a pivotal role in the successful design and implementation of this autonomous vehicle using Raspberry Pi. This project would not have been possible without their unwavering support and mentorship.

## Prerequisites

### Hardware
- **TurtleBot3 Burger**
- **Remote PC** with ROS 1 Noetic installed
- **Raspberry Pi Camera Module**
- **AutoRace Tracks and Objects** (Download from [ROBOTIS_GIT/autorace](https://github.com/ROBOTIS-GIT/autorace))

## Setup Instructions

### 1. Install AutoRace Packages

#### Remote PC
```sh
cd ~/catkin_ws/src/
git clone -b noetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_autorace_2020.git
cd ~/catkin_ws && catkin_make
sudo apt install ros-noetic-image-transport ros-noetic-cv-bridge ros-noetic-vision-opencv python3-opencv libopencv-dev ros-noetic-image-proc
```

#### SBC
```sh
cd ~/catkin_ws/src/
git clone -b feature-raspicam https://github.com/ROBOTIS-GIT/turtlebot3_autorace_2020.git
cd ~/catkin_ws && catkin_make
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo apt-get update
sudo apt-get install build-essential cmake gcc g++ git unzip pkg-config
sudo apt-get install libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libgtk2.0-dev libcanberra-gtk* libxvidcore-dev libx264-dev python3-dev python3-numpy python3-pip libtbb2 libtbb-dev libdc1394-22-dev libv4l-dev v4l-utils libopenblas-dev libatlas-base-dev libblas-dev liblapack-dev gfortran libhdf5-dev libprotobuf-dev libgoogle-glog-dev libgflags-dev protobuf-compiler
```

Build OpenCV:
```sh
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.0.zip
unzip opencv.zip
unzip opencv_contrib.zip
mv opencv-4.5.0 opencv
mv opencv_contrib-4.5.0 opencv_contrib
cd opencv
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules -D ENABLE_NEON=ON -D BUILD_TIFF=ON -D WITH_FFMPEG=ON -D WITH_GSTREAMER=ON -D WITH_TBB=ON -D BUILD_TBB=ON -D BUILD_TESTS=OFF -D WITH_EIGEN=OFF -D WITH_V4L=ON -D WITH_LIBV4L=ON -D WITH_VTK=OFF -D OPENCV_ENABLE_NONFREE=ON -D INSTALL_C_EXAMPLES=OFF -D INSTALL_PYTHON_EXAMPLES=OFF -D BUILD_NEW_PYTHON_SUPPORT=ON -D BUILD_opencv_python3=TRUE -D OPENCV_GENERATE_PKGCONFIG=ON -D BUILD_EXAMPLES=OFF ..
make -j4
sudo make install
sudo ldconfig
make clean
sudo apt-get update
```

Configure the Raspberry Pi camera:
- Edit `config.txt` on the microSD card and add `start_x=1` before `enable_uart=1`.
- Install FFmpeg:
```sh
sudo apt install ffmpeg
ffmpeg -f video4linux2 -s 640x480 -i /dev/video0 -ss 0:0:2 -frames 1 capture_test.jpg
```

Install additional dependent packages:
```sh
sudo apt install ros-noetic-cv-camera
```

### 2. Camera Calibration

#### Intrinsic Calibration
1. Launch `roscore` on Remote PC:
```sh
roscore
```
2. Trigger the camera on SBC:
```sh
roslaunch turtlebot3_autorace_camera raspberry_pi_camera_publish.launch
```
3. Run intrinsic camera calibration on Remote PC:
```sh
roslaunch turtlebot3_autorace_camera intrinsic_camera_calibration.launch mode:=calibration
```
4. Save calibration data to `camerav2_320x240_30fps.yaml`.

#### Extrinsic Calibration
1. Launch `roscore` on Remote PC:
```sh
roscore
```
2. Trigger the camera on SBC:
```sh
roslaunch turtlebot3_autorace_camera raspberry_pi_camera_publish.launch
```
3. Run extrinsic camera calibration on Remote PC:
```sh
roslaunch turtlebot3_autorace_camera extrinsic_camera_calibration.launch mode:=calibration
```
4. Use `rqt` and `rqt_reconfigure` to adjust parameters and save to YAML files.

### 3. Lane Detection

#### Calibration and Configuration
1. Place TurtleBot3 on the track.
2. Launch `roscore` on Remote PC:
```sh
roscore
```
3. Trigger the camera on SBC:
```sh
roslaunch turtlebot3_autorace_camera raspberry_pi_camera_publish.launch
```
4. Launch lane detection nodes on Remote PC:
```sh
roslaunch turtlebot3_autorace_detect detect_lane.launch mode:=calibration
```
5. Open `rqt` and launch `rqt_reconfigure`:
```sh
rqt
rosrun rqt_reconfigure rqt_reconfigure
```
6. Adjust parameters in `detect_lane` for line color filtering.

#### Lane Following Operation
1. Start lane following:
```sh
roslaunch turtlebot3_autorace_driving turtlebot3_autorace_control_lane.launch
```
2. Verify results using:
```sh
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

## Author
MOHD IBRAHIM KHAN  
ibrahimkafeel9@gmail.com

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
![IMG-20240503-WA0001](https://github.com/Integral-Robotics-Lab/Autonomous-Lane-Tracking/assets/146702742/a8b2e3cc-498a-433a-abae-b8e641918a15)
