# GndNet: Fast Ground plane Estimation and Point Cloud Segmentation for Autonomous Vehicles.
Authors: Anshul Paigwar, Ozgur Erkent, David Sierra Gonzalez, Christian Laugier

<img src="https://github.com/anshulpaigwar/GndNet/blob/master/doc/GndNet_Teaser.png" alt="drawing" width="400"/>

## Introduction

This repository is code release for our GndNet paper accepted in International conference on Robotic Systems, IROS 2020.

## Abstract

Ground plane estimation and ground point seg-mentation is a crucial precursor for many applications in robotics and intelligent vehicles like navigable space detection and occupancy grid generation, 3D object detection, point cloud matching for localization and registration for mapping. In this paper, we present GndNet, a novel end-to-end approach that estimates the ground plane elevation information in a grid-based representation and segments the ground points simultaneously in real-time. GndNet uses PointNet and Pillar Feature Encoding network to extract features and regresses ground height for each cell of the grid. We augment the SemanticKITTI dataset to train our network. We demonstrate qualitative and quantitative evaluation of our results for ground elevation estimation and semantic segmentation of point cloud. GndNet establishes a new state-of-the-art, achieves a run-time of 55Hz for ground plane estimation and ground point segmentation. 
<img src="https://github.com/anshulpaigwar/GndNet/blob/master/doc/GndNet_architecture_final.png" alt="drawing" width="800"/>

## Installation

We have tested the algorithm on the system with Ubuntu 18.04, 12 GB RAM and NVIDIA GTX-1080.

### Dependencies
```
Python 3.6
CUDA (tested on 10.1)
PyTorch (tested on 1.4)
scipy
ipdb
argparse
numba
```
### Visualization
For visualisation of the ground estimation, semantic segmentation of pointcloud, and easy integration with our real system we use Robot Operating System (ROS):
```
ROS
ros_numpy
```
## Data Preparation

* We train our model on augmented SematicKITTI dataset [link](http://www.semantic-kitti.org/).
* We subdivide object classes in SematicKITTI dataset into two categories 
	1. Ground (road, sidewalk, parking, other-ground, vegetation, terrain)
	2. Non-ground (all other)
* To prepare our ground elevation dataset we take only ground points and use the CRF-based surface fitting method described in [1].
* Ground labels are generated in a 2D grid representation with cell resolution 1m x 1m and of size (x, y) = [(-50, -50), (50, 50)]. Values of each cell represent the local ground elevation.
* We store ground labels and raw point clouds (both ground and non-ground points) to train our network.
* We provide a sample dataset in this repository, the full dataset can be made available on request.

## Training

To train the model update the data directory path in the config file: config_kittiSem.yaml
```
python main.py -s
```
 It takes around 6 hours for the network to converge and model parameters would be stored in checkpoint.pth.tar file. A pre-trained model is provided in the trained_models folder it can be used to evaluate a sequence in the SemanticKITTI dataset.

```
python evaluate_SemanticKITTI.py --resume checkpoint.pth.tar --data_dir /home/.../kitti_semantic/dataset/sequences/07/
```

## Using pre-trained model
Download the SemanticKITTI dataset from their website [link](http://www.semantic-kitti.org/). To visualize the output we use ROS and rviz. The predicted class (ground or non-ground) of the points in the point cloud is substituted in the intensity field of sensor_msgs.pointcloud. In the rviz use intensity as a color transformer to visualize segmented pointcloud. For the visualization of ground elevation, we use the ROS line marker. 

```
roscore
rviz
python evaluate_SemanticKITTI.py --resume trained_models/checkpoint.pth.tar -v -gnd --data_dir /home/.../SemanticKITTI/dataset/sequences/00/
```
Note: The current version of the code for visualization is written in python which can be very slow specifically the generation of ROS marker.
To only visualize segmentation output without ground elevation remove the `-gnd` flag.

## Results

Semantic segmentation of point cloud ground (green) and non-ground (purple):

<img src="https://github.com/anshulpaigwar/GndNet/blob/master/doc/segmntation_results.png" alt="drawing" width="800"/>

Ground elevation estimation:

<img src="https://github.com/anshulpaigwar/GndNet/blob/master/doc/ground_estimation.png" alt="drawing" width="800"/>

YouTube video (click on img):

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/kjZ-n_aIJAg/0.jpg)](https://www.youtube.com/watch?v=kjZ-n_aIJAg)

## TODO
* Current dataloader loads the entire dataset into RAM first, this reduces training time but it can be hog systems with low RAM.
* Speed up visualization of ground elevation. Write C++ code for ROS marker.
* Create generalized ground elevation dataset to be with correspondence to SemanticKitti to be made public.


## Citation

If you find this project useful in your research, please consider citing our work:
```
@inproceedings{paigwar2020gndnet,
  title={GndNet: Fast Ground Plane Estimation and Point Cloud Segmentation for Autonomous Vehicles},
  author={Paigwar, Anshul and Erkent, {\"O}zg{\"u}r and Gonz{\'a}lez, David Sierra and Laugier, Christian},
  booktitle={IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  year={2020}
}
```

## Contribution

We welcome you for contributing to this repo, and feel free to contact us for any potential bugs and issues.


## References

[1] L. Rummelhard, A. Paigwar, A. Nègre and C. Laugier, "Ground estimation and point cloud segmentation using SpatioTemporal Conditional Random Field," 2017 IEEE Intelligent Vehicles Symposium (IV), Los Angeles, CA, 2017, pp. 1105-1110, doi: 10.1109/IVS.2017.7995861.

[2] Behley, J., Garbade, M., Milioto, A., Quenzel, J., Behnke, S., Stachniss, C., & Gall, J. (2019). SemanticKITTI: A dataset for semantic scene understanding of lidar sequences. In Proceedings of the IEEE International Conference on Computer Vision (pp. 9297-9307).
