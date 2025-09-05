[English](./README.md) | 简体中文 

# Feature Introduction

The **palm_detection_mediapipe** package is a monocular RGB palm detection algorithm example developed using the **hobot_dnn** package.  
It runs on the RDK series development boards, leveraging the BPU processor for model inference with models and image data.  
The model output includes palm detection bounding boxes and palm root keypoint detection results.

The example subscribes to image data (`image msg`) and publishes customized perception results (`hobot ai msg`).  
Users can subscribe to the published `ai msg` for application development.

# Model and Platform Support

| Model Type | Supported Platform |
| :--------- | ------------------ |
| mediapipe  | RDK S100           |

# Bill of Materials

| Item Name  | Manufacturer | Reference Link |
| :--------- | ------------ | ---------------|
| RDK S100   | Multiple     | [RDK S100](https://developer.horizon.cc/rdks100) |
| Camera     | Multiple     | [MIPI Camera](https://developer.horizon.cc/nodehubdetail/168958376283445781)<br>[USB Camera](https://developer.horizon.cc/nodehubdetail/168958376283445777)|

# Preparation

- RDK has been flashed with Ubuntu 20.04/22.04 system image.
- Camera correctly connected to RDK S100.

# Usage

**1. Install the package**

After powering on the robot, connect to it via SSH terminal or VNC.  
Click the "One-click Deployment" button on the top right of this page, then copy the following command and run it on the RDK system to install the related Node.

```bash
sudo apt update
sudo apt install -y tros-humble-mono2d-palm-detection
```

**2. Run mediapipe palm detection**

**Using MIPI Camera to publish images**

```shell
# Configure tros humble environment
source /opt/tros/humble/setup.bash

# Copy the config files needed for running the example from the tros installation path
cp -r /opt/tros/${TROS_DISTRO}/lib/palm_detection_mediapipe/config/ .

# Configure MIPI camera
export CAM_TYPE=mipi

# Launch the file
ros2 launch palm_detection_mediapipe palm_detection.launch.py
```

**Using USB Camera to publish images**

```shell
# Configure tros humble environment
source /opt/tros/humble/setup.bash

# Copy the config files needed for running the example from the tros installation path
cp -r /opt/tros/${TROS_DISTRO}/lib/palm_detection_mediapipe/config/ .

# Configure USB camera
export CAM_TYPE=usb

# Launch the file
ros2 launch palm_detection_mediapipe palm_detection.launch.py
```

**Using Local Playback Images**

Only supported in tros humble version.

```shell
# Copy the config files needed for running the example from the tros installation path
cp -r /opt/tros/${TROS_DISTRO}/lib/palm_detection_mediapipe/config/ .

# Configure local playback images
export CAM_TYPE=fb

# Launch the file
ros2 launch palm_detection_mediapipe palm_detection.launch.py publish_image_source:=config/example.jpg publish_image_format:=jpg publish_output_image_w:=640 publish_output_image_h:=480
```

**4. View the results**

Open a browser on a computer in the same network and visit http://IP:8000

to see real-time visual recognition results, where IP is the RDK's IP address:

# Interface Description

## Topics

Palm detection results are published via the topic hobot_msgs/ai_msgs/msg/PerceptionTargets. The detailed definition of this topic is as follows:

```shell
# Perception result

# Message header
std_msgs/Header header

# Processing FPS of perception results
# fps val is invalid if fps is less than 0
int16 fps

# Performance statistics, e.g., inference time for each model
Perf[] perfs

# Detected targets
Target[] targets
```


| Name                 | Message Type        | Description|
| ---------------------- | ----------- |---------------------------- |
| /palm_detection_mediapipe          | [hobot_msgs/ai_msgs/msg/PerceptionTargets](https://github.com/D-Robotics/hobot_msgs/blob/develop/ai_msgs/msg/PerceptionTargets.msg)   | Publishes detected human target information |
| /hbmem_img | [hobot_msgs/hbm_img_msgs/msg/HbmMsg1080P](https://github.com/D-Robotics/hobot_msgs/blob/develop/hbm_img_msgs/msg/HbmMsg1080P.msg)  | When is_shared_mem_sub == 1, subscribes to image data via shared memory|
| /image_raw | hsensor_msgs/msg/Image  |  When is_shared_mem_sub == 0, subscribes to image data via standard ROS|



## Parameters

| Parameter Name            | Type        | Description                                                                                                               | Required | Supported Values     | Default Value                   |
| ------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------- | -------- | -------------------- | ------------------------------- |
| is\_sync\_mode            | int         | Sync/Async inference mode. `0`: async mode; `1`: sync mode.                                                               | No       | 0/1                  | 0                               |
| model\_file\_name         | std::string | Model file used for inference.                                                                                            | No       | Based on actual path | config/palm\_det\_192\_192.hbm  |
| is\_shared\_mem\_sub      | int         | Whether to use shared memory to subscribe to image messages. `0`: off; `1`: on. Topics are `/hbmem_img` and `/image_raw`. | No       | 0/1                  | 1                               |
| ai\_msg\_pub\_topic\_name | std::string | Topic name for publishing AI messages containing keypoints detection results.           | No       | Configurable         | /hobot\_mono2d\_palm\_detection |
| ros\_img\_topic\_name     | std::string | ROS image topic name                                                                                                      | No       | Configurable         | /image\_raw                     |
| image\_gap                | int         | Frame skipping interval. `1`: process every frame, `2`: process every two frames, etc.                                    | No       | Configurable         | 1                               |