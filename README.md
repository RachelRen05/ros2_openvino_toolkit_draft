# Introduction

The OpenVINO™ (Open visual inference and neural network optimization) toolkit provides a ROS-adaptered runtime framework of neural network which quickly deploys applications and solutions for vision inference. By leveraging Intel® OpenVINO™ toolkit and corresponding libraries, this runtime framework extends  workloads across Intel® hardware (including accelerators) and maximizes performance.
* Enables CNN-based deep learning inference at the edge
* Supports heterogeneous execution across computer vision accelerators—CPU, GPU, Intel® Movidius™ Neural Compute Stick, and FPGA—using a common API
* Speeds up time to market via a library of functions and preoptimized kernels
* Includes optimized calls for OpenCV and OpenVX*

## Design Architecture
From the view of hirarchical architecture design, the package is divided into different functional components, as shown in below picture. 

![OpenVINO_Architecture](https://github.com/LewisLiuPub/ros2_openvino_toolkit/blob/guide-info/doc/design_arch.PNG "OpenVINO RunTime Architecture")

- **Intel® OpenVINO™ toolkit** is leveraged to provide deep learning basic implementation for data inference. is free software that helps developers and data scientists speed up computer vision workloads, streamline deep learning inference and deployments,
and enable easy, heterogeneous execution across Intel® platforms from edge to cloud. It helps to:
   - Increase deep learning workload performance up to 19x1 with computer vision accelerators from Intel.
   - Unleash convolutional neural network (CNN)-based deep learning inference using a common API.
   - Speed development using optimized OpenCV* and OpenVX* functions.
- **ROS2 OpenVINO Runtime Framework** is the main body of this repo. it provides key logic implementation for pipeline lifecycle management, resource management and ROS system adapter, which extends Intel OpenVINO toolkit and libraries. Furthermore, this runtime framework provides ways to ease launching, configuration and data analytics and re-use.
- **Diversal Input resources** are the data resources to be infered and analyzed with the OpenVINO framework.
- **ROS interfaces and outputs** currently include _Topic_ and _service_. Natively, RViz output and CV image window output are also supported by refactoring topic message and inferrence results.
- **Optimized Models** provides by Model Optimizer component of Intel® OpenVINO™ toolkit. Imports trained models from various frameworks (Caffe*, Tensorflow*, MxNet*, ONNX*, Kaldi*) and converts them to a unified intermediate representation file. It also optimizes topologies through node merging, horizontal fusion, eliminating batch normalization, and quantization.It also supports graph freeze and graph summarize along with dynamic input freezing.

## Logic Flow
From the view of logic implementation, the package introduces the definitions of parameter manager, pipeline and pipeline manager. The below picture depicts how these entities co-work together when the corresponding program is launched.

![Logic_Flow](https://github.com/LewisLiuPub/ros2_openvino_toolkit/blob/guide-info/doc/impletation_logic.PNG "OpenVINO RunTime Logic Flow")

Once a corresponding program is launched with a specified .yaml config file passed in the .launch.py file or via commandline, _**parameter manager**_ analyzes the configurations about pipeline and the whole framework, then shares the parsed configuration information with pipeline procedure. A _**pipeline instance**_ is created by following the configuration info and is added into _**pipeline manager**_ for lifecycle control and inference action triggering.

The contents in **.yaml config file** should be well structured and follow the supported rules and entity names. Please see [the configuration guidance](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/doc/YAML_CONFIGURATION_GUIDE.MD) for how to create or edit the config files.

**Pipeline** fulfills the whole data handling process: initiliazing Input Component for image data gathering and formating; building up the structured inference network and passing the formatted data through the inference network; transfering the inference results and handling output, etc.

**Pipeline manager** manages all the created pipelines according to the inference requests or external demands (say, system exception, resource limitation, or end user's operation). Because of co-working with resource management and being aware of the whole framework, it covers the ability of performance optimization by sharing system resource between pipelines and reducing the burden of data copy.

# Supported Features
## Diversal Input Components
Currently, the package support several kinds of input resources of gaining image data:

|Input Resource|Description|
|--------------------|------------------------------------------------------------------|
|StandardCamera|Any RGB camera with USB port supporting. Currently only the first USB camera if many are connected.|
|RealSenseCamera| Intel RealSense RGB-D Camera, directly calling RealSense Camera via librealsense plugin of openCV.|
|Image Topic| any ROS topic which is structured in image message.|
|Image File| Any image file which can be parsed by openCV, such as .png, .jpeg.|
|Video File| Any video file which can be parsed by openCV.|

## Inference Implementations
Currently, the inference feature list is supported:

|Inference|Description|
|-----------------------|------------------------------------------------------------------|
|Face Detection|Object Detection task applied to face recognition using a sequence of neural networks.|
|Emotion Recognition| Emotion recognition based on detected face image.|
|Age & Gender Recognition| Age and gener recognition based on detected face image.|
|Head Pose Estimation| Head pose estimation based on detected face image.|
|Object Detection| object detection based on SSD-based trained models.|
|Vehicle Detection| Vehicle and passenger detection based on Intel models.|
|Object Segmentation| object detection and segmentation.|

## ROS interfaces and outputs
### Topic
#### Subscribed Topic
```/camera/color/image_raw```([sensor_msgs::msg::Image](https://github.com/ros2/common_interfaces/blob/master/sensor_msgs/msg/Image.msg))
#### Published Topic
- Face Detection:
```/ros2_openvino_toolkit/face_detection```([object_msgs:msg:ObjectsInBoxes](https://github.com/intel/ros2_object_msgs/blob/master/msg/ObjectsInBoxes.msg))
- Emotion Recognition:
```/ros2_openvino_toolkit/emotions_recognition```([people_msgs:msg:EmotionsStamped](https://github.com/intel/ros2_openvino_toolkit/blob/master/people_msgs/msg/EmotionsStamped.msg))
- Age and Gender Recognition:
```/ros2_openvino_toolkit/age_genders_Recognition```([people_msgs:msg:AgeGenderStamped](https://github.com/intel/ros2_openvino_toolkit/blob/master/people_msgs/msg/AgeGenderStamped.msg))
- Head Pose Estimation:
```/ros2_openvino_toolkit/headposes_estimation```([people_msgs:msg:HeadPoseStamped](https://github.com/intel/ros2_openvino_toolkit/blob/master/people_msgs/msg/HeadPoseStamped.msg))
- Object Detection:
```/ros2_openvino_toolkit/object_detection```([object_msgs::msg::ObjectsInBoxes](https://github.com/intel/ros2_object_msgs/blob/master/msg/ObjectsInBoxes.msg))
- Object Segmentation:
```/ros2_openvino_toolkit/segmented_obejcts```([people_msgs::msg::ObjectsInMasks](https://github.com/intel/ros2_openvino_toolkit/blob/devel/people_msgs/msg/ObjectsInMasks.msg))
- Rviz Output:
```/ros2_openvino_toolkit/image_rviz```([sensor_msgs::msg::Image](https://github.com/ros2/common_interfaces/blob/master/sensor_msgs/msg/Image.msg))

### Service
TBD

### RViz
RViz dispaly is also supported by the composited topic of original image frame with inference result.
To show in RViz tool, add an image marker with the composited topic:
```/ros2_openvino_toolkit/image_rviz```([sensor_msgs::msg::Image](https://github.com/ros2/common_interfaces/blob/master/sensor_msgs/msg/Image.msg))

### Image Window
OpenCV based image window is natively supported by the package.
To enable window, Image Window output should be added into the output choices in .yaml config file. see [the config file guidance](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/doc/YAML_CONFIGURATION_GUIDE.MD) for checking/adding this feature in your launching.

## Running the Demo
* Preparation
	* download and convert a trained model to produce an optimized Intermediate Representation (IR) of the model 
		```bash
		sudo mkdir -p ~/Downloads/models
		cd ~/Downloads/models
		wget http://download.tensorflow.org/models/object_detection/mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
		tar -zxvf mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
		cd mask_rcnn_inception_v2_coco_2018_01_28
		python3 /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/mo_tf.py --input_model frozen_inference_graph.pb --tensorflow_use_custom_operations_config /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/extensions/front/tf/mask_rcnn_support.json --tensorflow_object_detection_api_pipeline_config pipeline.config --reverse_input_channels --output_dir ./output/
		sudo mkdir -p /opt/models
		sudo ln -s ~/Downloads/models/mask_rcnn_inception_v2_coco_2018_01_28 /opt/models/
	* download the optimized Intermediate Representation (IR) of model (excute _once_)<br>
		```bash
		cd /opt/openvino_toolkit/open_model_zoo/model_downloader
		python3 downloader.py --name face-detection-adas-0001
		python3 downloader.py --name age-gender-recognition-retail-0013
		python3 downloader.py --name emotions-recognition-retail-0003
		python3 downloader.py --name head-pose-estimation-adas-0001
		python3 downloader.py --name ssd300
		```
	* copy label files (excute _once_)<br>
		```bash
		sudo cp /opt/openvino_toolkit/ros2_openvino_toolkit/data/labels/emotions-recognition/FP32/emotions-recognition-retail-0003.labels /opt/openvino_toolkit/open_model_zoo/model_downloader/Retail/object_attributes/emotions_recognition/0003/dldt
		sudo cp /opt/openvino_toolkit/ros2_openvino_toolkit/data/labels/object_segmentation/frozen_inference_graph.labels ~/Downloads/models/mask_rcnn_inception_v2_coco_2018_01_28/output
		```
	* set ENV LD_LIBRARY_PATH<br>
		```bash
		export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openvino_toolkit/dldt/inference-engine/bin/intel64/Release/lib
		```
* run face detection sample code input from StandardCamera.
	```bash
	ros2 launch dynamic_vino_sample pipeline_people.launch.py
	```
* run face detection sample code input from Image.
	```bash
	ros2 launch dynamic_vino_sample pipeline_image.launch.py
	```
![face_detection_demo_image](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/data/images/face_detection.png "face detection demo image")
* run object detection sample code input from RealSenseCamera.
	```bash
	ros2 launch dynamic_vino_sample pipeline_object.launch.py
	```
![object_detection_demo_realsense](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/data/images/object_detection.gif "object detection demo realsense")
* run object detection sample code input from RealSenseCameraTopic.
	```bash
	ros2 launch dynamic_vino_sample pipeline_video.launch.py
	```
* run object segmentation sample code input from Video.
	```bash
	ros2 launch dynamic_vino_sample pipeline_segmentation.launch.py
	```
	![object_segmentation_demo_video](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/data/images/object_segmentation.gif "object segmentation demo video")

# Installation & Launching
## Dependencies Installation
One-step installation scripts are provided for the dependencies' installation. Please see [the guide](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/doc/OPEN_SOURCE_CODE_README.md) for details.
Note that beside the opensource version of the openVINO toolkit, Intel also releases the toolkit in Binary version. If you prefer to use the binary version, you may follow [this guide](https://github.com/RachelRen05/ros2_openvino_toolkit_draft/blob/master/doc/BINARY_VERSION_README.md).



# TODO Features

# More Information

