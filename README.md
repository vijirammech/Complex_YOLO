# ComplexYOLO
This repository contains a PyTorch implementation of [ComplexYOLO](https://arxiv.org/pdf/1803.06199.pdf). It is build to be applied on the data from the Astyx Dataset.

## Installation
#### Clone the project and install requirements
    $ git clone https://github.com/BerensRWU/ComplexYOLO/
    
### Requirements
The script was tested on the following systems with the following versions:
#### Ubuntu 18.04.5 LTS
To run the program you need to install those libraries with dependencies:
  * torch (Tested with pytorch version 1.6.0, torchvision 0.7.0)
  * cv2 
  * json
  * numpy 
  * shapely 
  
  
#### Windows 10
To run the program you need to install those libraries with dependencies:
  * torch (Tested with pytorch version 1.5.1, torchvision 0.6.1)
  * cv2 
  * json
  * numpy 
  * shapely 
  
An easy way to install this is with [anaconda](https://www.anaconda.com/) or [miniconda](https://docs.conda.io/en/latest/miniconda.html). Then you can install e.g. torch:
```
conda install pytorch torchvision cpuonly -c pytorch
```
  
## Skripts
main.py.
```
python main.py
```
#### Arguments:
  * ```estimate_bb```: Flag, to estimate bounding boxes. Default ```False```.
  * ```radar```: Flag, if True use radar data otherwise lidar. Default ```False```.
  * ```model_def```: Path to model definition file. Default ```config/yolov3-custom.cfg```.
  * ```weights_path```: Path where the weights are saved. Default ```checkpoints```.
  * ```conf_thres```: If bounding boxes are estimated, only those with a confidence greater than thethreshold will be used. Default ```0.5```.
  * ```iou_thres```: If bounding boxes are estimated, threshold for a correct detection. Default ```0.5```.
  * ```nms_thres```: If estimated bounding boxes overlap with an IoU greater than the ```nms_thres``` only the bounding box with highest confidence remains. Default ```0.2```.
  * ```split```: Which split to use ```valid```, ```train```. Default ```valid```.
  * ```visualize```: Whether to visualize the data.
  * ```evaluate```: Whether to evaluate the data.
  
### visualize.py
A script for the visualization of the data.

### detector.py
A script for the detection of bounding boxes from BEV.

### evaluation.py
A script for the evaluation of the predicted bounding boxes. Here we need to calculate whether the prediction is a true positive or not and the average precision.

# How to use this Repo for the LiDAR RADAR lecture
If you want to use this repository for the LiDAR RADAR lecture, you have two ways:
## Terminal
To run on your own machine in the terminal, download this repository. Install python and all requirements. Define the path to the data in utils/config.py it should have the order like in section Astyx HiRes. Now we can run the main function in the terminal:
```
python main.py
```
This will check if you have defined the correct path to the data.

In the next step we will visualize the data:
```
python main.py --visualize
```
This will save the BEVs and the ground truth data in the folder ```output```. In default this will use LiDAR data, if we want to use radar data we have to add the flag ```--radar```. The coordinates of the ground truth are in the RADAR coordinatesystem. Because of this the LiDAR data is transformed to the RADAR coordinatesystem.

Next we want to predict bounding boxes. For this we need to download the weights for the network and store them in the folder ```checkpoints```, one for RADAR and one for LiDAR. Also we need non maximum supression, such that predicted bounding boxes that overlap get merged to one. For the non maximum supression we need the intersection over union, to write this function is your task. You find a starting point in ```utils/utils.py``` in the function ```def compute_iou(box, boxes)``` it gets one bounding box as shapley polygon and a list of shapley polygones and returns a list of the corresponding iou values. Then the prediction can be done with the following command:
```
python main.py --estimate_bb
```
To visualize and predict we can use the following command:
```
python main.py --estimate_bb --visualize
```
To evaluate the predictions we want to use the average precision, so we need next to a function for the IoU a function for the average precision. In the script ```evaluation.py``` you find funcitons for this, which you have to complete:
```get_batch_statistics_rotated_bbox(predictions, targets, iou_threshold)``` calculates whether a prediction is a correct prediction (True positive) or not. The input of this function are all predictions, the targets and the IoU threshold. You will start after the calculation of the IoUs between. In the function we loop over the batch and then over the predictions. So the IoU is calculated between one prediction and all ground truth bounding boxes. Now you have to code:

Get the maximum IoU of this prediction and get the corresponding target id. Then check if this IoU is greater than the IoU threshold and if we have not already detect this ground truth object. If it is greater and not already detected, then we store in ```true_positives``` at the index of the prediction a True or 1 and also append to ```detected_boxes``` the index of the ground truth object. Then return the list ```true_positives``` and list of the score values(```pred_scores```).

After this we have to concatenate the lists to one long list for all validation frames. This long list is used for the calculation of the AP in calculate_ap. This function gets the two long lists true_positives, pred_scores and the number of all ground truth objects. To calculate the AP we can do the following steps as known from the lecture:
1. Calculate a list of False positves (1 - true positves)
1. Sort the lists according to the scores
1. Calculate the Cumulative sums of the sorted Lists
1. Calculate the precision and recall for every entry of the cumulative sums
    precision = cumsumTP/(cumsumTP + cumsumFP)
    recall = cumsumTP / ngt
1. Concatenate 0 / 1 to the start/ending of the precision list
   Concatenate 0 / 0 to the start/ending of the recall list
1. Calculate the approximated AP as described in the Lecture ( AP-11, AP-all)

## Colab
If you want to use colab you click on "main.ipynb" and then on "open in colab". After you locked in your google account, you have to upload the scripts to the  files in the notebook. Because of the size the Astyx data should be uploaded to your drive and change in utils->config->root_dir to "drive/My Drive/dataset" or wherever you saved the dataset. Instead of flags we can specify in the fourth cell what we want to do. The rest are the same steps as for the terminal way.

# Astyx HiRes
The Astyx HiRes is a dataset from Astyx for object detection for autonomous driving. Astyx has a sensor setup consisting of camera, LiDAR, RADAR. Additional information can be found here: [Dataset Paper](https://www.astyx.com/fileadmin/redakteur/dokumente/Automotive_Radar_Dataset_for_Deep_learning_Based_3D_Object_Detection.PDF) and [Specification](https://www.astyx.com/fileadmin/redakteur/dokumente/Astyx_Dataset_HiRes2019_specification.pdf)

```
└── dataset/
       ├── dataset_astyx_hires2019    <-- 546 data
       |   ├── calibration 
       |   ├── camera_front
       |   ├── groundtruth_obj3d
       |   ├── lidar_vlp16
       |   ├── radar_6455
       └── split
           ├── train.txt
           └── valid.txt
```
# Evaluation
To evaluate the valid split we need a function for IoU and AP. For an confidence threshold of 0.5, non-maximum-supression threshold of 0.2 and a IoU threshold of 0.5 we get an average precision (AP) in the range of:

 Model - Sensor/Class | Car     | 
| ------------------- |:--------|
| ComplexYOLO LiDAR   | 65 - 75 |
| ComplexYOLO RADAR   | 65 - 75 |
