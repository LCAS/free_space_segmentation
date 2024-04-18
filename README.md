# free_space_segmentation
The project is founded on the precedent set by the following project ["ORFD: Off-Road-Freespace-Detection
"](https://github.com/chaytonmin/Off-Road-Freespace-Detection/tree/main).

## Project Overview 

The ORFD project has been resumed to test its implementation in detecting pathways in polytunnels dedicated to strawberry harvesting. The repository with modifications to the original code can be found here: [Off-road-detection](https://github.com/adri-gth/Off-road-detection/tree/main), The generated dataset includes a total of 34,684 data and is available at [polytunnel_dataset](https://drive.google.com/file/d/1egS08WVoOzbN0vwSiknT8DT6aaipX_3V/view?usp=sharing)


**Project Structure**

The project is divided into three sections:

- Data Collection
 
- Generation of the polytunnel_dataset
 
- Testing the OFF-Net model trained with the ORFD dataset

**Repository Contents**

- The *free_space_segmentation* folder contains the package created for extracting information from bag files.
  
- The *docker* folder includes a shell script, the usage of which is explained later.

- The *python_code* folder contains the code needed to extract the ground truth images (*gt_image*).


### Setting Up the Work Environment

#### Instruction for running the docker image

The *docker* folder contains a shell file. This file must be downloaded to the host and executed using the following commands:
 
 To start the container for the first time, execute the following command:
 
    cd <the directory where the obs_detect.sh file is located> 
    ./obs_detect.sh run
     
This command sets up and runs a new container using the 'adriavdocker/obsdetection:v2' image 

To start the container that has already been created: 

    ./obs_detect.sh start
    
To enter the container after it has been started

    ./obs_detect.sh enter
    
To stop the container 

    ./obs_detect.sh stop
    
Additional Notes: Ensure that the *obs_detect.sh* script has execute permissions. If it does not, you can grant execute permissions using the following command:

    chmod +x obs_detect.sh


## Data Collection 

### Physical Setup of Sensors on the Robot

To collect data, two sensors are mounted on a structure located atop a [Hunter 2.0](https://docs.trossenrobotics.com/agilex_hunter_20_docs/). The first is a LiDAR [LIVOX-MID360](https://www.livoxtech.com/mid-360) and the second is a [ZED 2 Stereo Camera](https://store.stereolabs.com/en-gb/products/zed-2). These sensors are stacked one above the other, as illustrated in the following diagram:
```
                                                                    _____LiDAR______
2.25 ft----------------- Camera height------------------------------__Depth_camera__
  |                                                                        |         
  |                                                                        |
  |                                                                        |
1.2 ft----------------- Hunter's max height--------------------------------|
  |                                                                        |
  |                                                                        |
  |                                                                        |
  |                                                                        |
__|__________________________Floor_________________________________________|_________________
```
### Extracting Depth images, RGB images and point clouds from a ROS2 topic in a synchronized manner.

**To obtain information in a synchronized manner (for the ORFD project, this is mandatory), the data_synchronizer node should be executed:**
 
    ros2 run free_space_segmentation sync_node 

The root path where the information is stored is:

- /home/data/synchronized_dataset/datasets/ORFD/testing/sequence/****

The folders created to store the information are:

    rgb_image: image_data 
    point Cloud: lidar_data 
    calib: calib 
    depth_image: dense_depth 

    
**Extracting RGB Images, Point Clouds, and Depth Images Individually**


To extract only the RGB images, execute the following command:

    ros2 run free_space_segmentation save_rgb_image

This command stores the RGB images and calibration data for the camera and LiDAR sensor.

The root path where the information is stored is:

- /home/data/unsynchronized_dataset/datasets/ORFD/testing/image_data
- /home/data/unsynchronized_dataset/datasets/ORFD/testing/calib

To extract only point clouds, execute the following command:

     ros2 run free_space_segmentation save_point_Cloud 

This command saves the point cloud generated by the LiDAR sensor as a .bin file. 

The root path where the information is stored is:

- /home/data/unsynchronized_dataset/datasets/ORFD/testing/sequence/lidar_data

To extract only depth images, execute the following command:

      ros2 run free_space_segmentation save_depth_image 

The root path where the information is stored is:

- /home/data/unsynchronized_dataset/datasets/ORFD/testing/sequence/dense_depth

### Instructions for creating a testing dataset

With the folders generated previously, the only thing left to do is to generate *gt_image*. To accomplish this, the [Supervisely](https://supervisely.com/) platform is used.

To generate the ground truth image data:

1. The *image_data* folder, generated by the sync_node , is uploaded to it.

2. In the platform Supervisely, the *road* class is created, applicable to *any shape*, which enables the segmentation of paths.

3. After the segmentation is applied to the data, the project is downloaded and exported using the *Export to Supervisely Format* option, retaining only the *.json* annotations.
    
4. Subsequently, in the *python_code* folder, there is an *extract_mask.py* script. In this script, the path to the file downloaded from Supervisely, the name of that file, and the folder in which the mask will be extracted need to be updated. The script creates the *gt_image* folder.


The folders *lidar_data*, *dense_depth*, *image_data*, *calib*, and *gt_image* are already available. They should be organized in the following manner:

```
|-- datasets
 |  |-- ORFD
 |  |  |-- training
 |  |  |  |-- sequence   |-- calib
 |  |  |                 |-- sparse_depth
 |  |  |                 |-- dense_depth
 |  |  |                 |-- lidar_data
 |  |  |                 |-- image_data
 |  |  |                 |-- gt_image
 ......
 |  |  |-- validation
 ......
 |  |  |-- testing
 ......
```
The location of datasets, which is the folder containing all the information, must be the same as the path for *test.sh* and *train.sh* so that the program can access the information.


## Testing the OFF-Net Model
The results obtained are presented in the following demo. The image on the left is the RGB image, the center image is a depth image, and the image on the right is a composite of the RGB image, the model's prediction, and a mask that highlights the model's predictions for navigable space in green.


<p align="center">
<img src="doc/demo.gif" width="100%"/>demo 
</p>





















