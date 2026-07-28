# ROS Notes for Embedded Vision (EST Preparation)

---

# ROS (Robot Operating System)

## Definition

ROS (Robot Operating System) is a middleware framework that enables different modules of a robot to communicate with each other. It is not an operating system itself but provides tools, libraries, and communication mechanisms for building robotic applications.

ROS is mainly used to integrate:
- Cameras
- Sensors
- AI Models
- Robot Motion
- Actuators
- Navigation

---

# Why ROS?

Advantages:
- Modular architecture
- Reusable nodes
- Easy integration of sensors and cameras
- Topic-based communication
- Distributed processing
- Easy debugging
- Scalable for complex robotic systems

---

# ROS Architecture

```text
              Camera
                 │
                 ▼
         Camera Node (Publisher)
                 │
                 ▼
        /camera/image_raw
                 │
                 ▼
      Processor Node (Subscriber)
        Object Detection /
         Segmentation /
         Classification
                 │
                 ▼
        /processed/image
                 │
                 ▼
         Display Node /
       image_view / Browser
```

---

# Core ROS Concepts

## 1. Node

A node is an executable program that performs one specific task.

Examples:
- Camera Node
- Processor Node
- Display Node

Advantages:
- Modular
- Independent
- Easy to debug

---

## 2. Topic

A topic is a communication channel through which nodes exchange messages.

Example:

```
/camera/image_raw
```

---

## 3. Publisher

A publisher sends data to a topic.

Example:

```
Camera Node
        │
Publishes images
        │
/camera/image_raw
```

---

## 4. Subscriber

A subscriber receives data from a topic.

Example:

```
Processor Node
        │
Subscribes to
/camera/image_raw
```

---

## 5. Message

Messages define the format of data exchanged between nodes.

Example:

```python
from sensor_msgs.msg import Image
```

---

## 6. Package

A package is the basic organizational unit in ROS.

It contains:
- Source code
- Nodes
- Launch files
- Configurations
- Dependencies

---

## 7. ROS Master

Started using:

```bash
roscore
```

Functions:
- Registers nodes
- Manages topics
- Enables communication
- Parameter server

Without roscore, nodes cannot communicate.

---

# ROS1 vs ROS2

| ROS1 | ROS2 |
|------|------|
| TCPROS communication | DDS communication |
| Limited real-time support | Better real-time support |
| Basic security | Built-in security |
| Older architecture | Modern architecture |
| Less scalable | Highly scalable |

---

# Sample Vision Pipeline

```
USB Camera
      │
      ▼
Camera Node
      │
      ▼
/camera/image_raw
      │
      ▼
Processor Node
(OpenCV/CUDA/AI)
      │
      ▼
/processed/image
      │
      ▼
Display Node
```

---

# ROS Python Code Explanation

## Imports

```python
import rospy
```

ROS Python API.

---

```python
from sensor_msgs.msg import Image
```

Imports Image message type.

---

```python
from cv_bridge import CvBridge
```

Converts:

```
ROS Image
      ↔
OpenCV Image
```

---

```python
import cv2
```

OpenCV library.

---

```python
import time
```

Measures processing time.

---

# Subscriber

```python
rospy.Subscriber(
"/camera/image_raw",
Image,
image_callback
)
```

Meaning:
- Subscribe to `/camera/image_raw`
- Receive Image messages
- Execute `image_callback()` whenever a frame arrives

---

# Publisher

```python
rospy.Publisher(
"/processed/image",
Image,
queue_size=1
)
```

Publishes processed images.

queue_size controls message buffering.

---

# CvBridge

ROS Image format cannot be processed directly by OpenCV.

Convert ROS → OpenCV

```python
bridge.imgmsg_to_cv2(msg,"bgr8")
```

Convert OpenCV → ROS

```python
bridge.cv2_to_imgmsg(img,"mono8")
```

---

# Callback Function

```python
def image_callback(msg):
```

Automatically executes whenever a new message arrives.

---

# GPU Processing

Upload image:

```python
gpu.upload(image)
```

Process using CUDA:

- Grayscale
- Gaussian Blur
- Canny Edge Detection

Download result:

```python
gpu.download()
```

---

# rospy.spin()

```python
rospy.spin()
```

Keeps node running continuously.

Without this, the node exits immediately.

---

# Camera Node

Purpose:
- Access camera
- Capture frames
- Convert OpenCV → ROS
- Publish images

Pipeline

```
USB Camera
      │
      ▼
OpenCV Frame
      │
      ▼
CvBridge
      │
      ▼
ROS Image
      │
      ▼
/camera/image_raw
```

---

# Processor Node

Purpose:
- Subscribe to camera topic
- Process image
- Publish processed image

Pipeline

```
/camera/image_raw
       │
       ▼
Subscriber
       │
       ▼
Image Processing
       │
       ▼
/processed/image
```

---

# Creating ROS Workspace

Create workspace

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
```

Create package

```bash
catkin_create_pkg dusty_inference rospy std_msgs sensor_msgs cv_bridge image_transport
```

---

# Package Dependencies

| Dependency | Purpose |
|------------|----------|
| rospy | ROS Python |
| std_msgs | Standard messages |
| sensor_msgs | Camera/Sensor messages |
| cv_bridge | ROS ↔ OpenCV |
| image_transport | Efficient image transfer |

---

# Build Workspace

```bash
cd ~/catkin_ws
catkin_make
```

Registers package

```bash
source devel/setup.bash
```

Always source after every successful build.

---

# Camera Drivers

USB Camera

```bash
sudo apt install ros-noetic-usb-cam
rosrun usb_cam usb_cam_node
```

Publishes:

```
/usb_cam/image_raw
```

---

V4L2 Camera

```bash
sudo apt install ros-noetic-v4l2-camera
rosrun v4l2_camera v4l2_camera_node
```

Publishes:

```
/camera/image_raw
```

---

# CMakeLists.txt

```cmake
catkin_install_python(PROGRAMS
src/camera_node.py
src/processor_node.py
DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

Purpose:
- Makes Python files executable ROS nodes.

Without this:

```bash
rosrun package_name node.py
```

will fail.

---

# Display Output

Install image viewer

```bash
sudo apt install ros-noetic-image-view
```

Run

```bash
rosrun image_view image_view image:=/processed/image
```

---

# Web Streaming

Install

```bash
sudo apt install ros-noetic-web-video-server
```

Run

```bash
rosrun web_video_server web_video_server
```

Browser

```
http://<jetson-ip>:8080/stream?topic=/processed/image
```

---

# Launch File

Instead of starting nodes individually:

```bash
roslaunch dusty_inference system.launch
```

Example

```xml
<launch>

<node pkg="v4l2_camera"
      type="v4l2_camera_node"
      name="camera_node"/>

<node pkg="dusty_inference"
      type="processor_node.py"
      name="processor_node"/>

<node pkg="web_video_server"
      type="web_video_server"
      name="web_video_server"/>

</launch>
```

Advantages:
- Starts all nodes together
- Cleaner deployment
- Easier management

---

# Topic Remapping

```xml
<remap
from="/usb_cam/image_raw"
to="/camera/image_raw"/>
```

Purpose:
- Change topic names without modifying code.

---

# ROS Deployment Flow

```
Start Docker
      │
      ▼
Create Catkin Workspace
      │
      ▼
Create Package
      │
      ▼
Write Nodes
      │
      ▼
Modify CMakeLists
      │
      ▼
catkin_make
      │
      ▼
source devel/setup.bash
      │
      ▼
roscore
      │
      ▼
Run Camera Node
      │
      ▼
Run Processor Node
      │
      ▼
Run image_view
```

---

# Common Build Errors

## Build Failed

```bash
rm -rf build devel
```

Rebuild

```bash
catkin_make
source devel/setup.bash
```

---

## cv_bridge Error

Clone manually

```bash
git clone https://github.com/ros-perception/vision_opencv.git
cd vision_opencv
git checkout noetic
```

---

## image_transport Error

```bash
git clone https://github.com/ros-perception/image_common.git
cd image_common
git checkout noetic-devel
```

---

## Boost Error

```bash
ln -s /usr/lib/aarch64-linux-gnu/libboost_python3.so libboost_python37.so
```

Purpose:
Create symbolic link to resolve library version mismatch.

---

## yaml-cpp Error

```bash
git clone https://github.com/jbeder/yaml-cpp.git

git checkout yaml-cpp-0.6.3

mkdir build
cd build

cmake .. -DCMAKE_POSITION_INDEPENDENT_CODE=ON

make -j4

sudo make install

sudo ldconfig

export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH
```

---

# Typical 9-Mark Answer Structure (ROS)

1. Define the problem.
2. Draw ROS architecture.
3. Identify required nodes.
4. Mention publishers and subscribers.
5. Mention topics used.
6. Explain image processing flow.
7. Explain package creation.
8. Explain launching using `roslaunch`.
9. Conclude with advantages of ROS.

---

# Frequently Asked Viva Questions

### Why use ROS?
Modular communication between robot components.

### What is a Node?
An executable program performing one task.

### What is a Topic?
Communication channel between ROS nodes.

### What is a Publisher?
Node that sends messages.

### What is a Subscriber?
Node that receives messages.

### What is CvBridge?
Converts ROS Images ↔ OpenCV Images.

### Why use catkin_make?
Builds ROS packages.

### Why source setup.bash?
Registers workspace with ROS.

### Why use launch files?
To start multiple nodes together.

### Why use image_transport?
Efficient image communication.

### Why use web_video_server?
Remote browser-based visualization.

### Why use topic remapping?
Change topic names without modifying source code.

---

# Complete ROS Vision Pipeline

```
Camera
   │
   ▼
Camera Node
(Publisher)
   │
   ▼
/camera/image_raw
   │
   ▼
Processor Node
(Object Detection /
 Segmentation /
 Classification)
   │
   ▼
/processed/image
   │
   ├──────────────► image_view
   │
   └──────────────► web_video_server
```
