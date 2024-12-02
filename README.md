# Object Detection For Autonomous Driving using YOLO algorithm

-----------------

Before starting to read this, I would like you first to download these two videos of 2Mo each :

[Dataset](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/videos/road_video.mp4)

[Prediction](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/videos/pred_video.mp4)

The dataset is made out of 120 images taken by a camera on the hood of a moving car to simulate what autonomous driving will look like.

it is provided by : 
![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/driveai.png?raw=true)

The prediction is the same video made out of the same 120 images after being fed to the model to draw the bounding boxes.

You can predict using your own photos, just put them in the images folder, and the pediction will be saved in the out folder using this cell : `out_scores, out_boxes, out_classes = predict(sess, "image_name.jpg")`

-----------------

YOLO ("you only look once") is a popular algoritm because it achieves high accuracy while also being able to run in real-time, almost clocking 45 frames per second. A smaller version of the network, Fast YOLO, processes an astounding 155 frames per second while still achieving double the mAP of other real-time detectors. This algorithm "only looks once" at the image in the sense that it requires only one forward propagation pass through the network to make predictions. After non-max suppression, it then outputs recognized objects together with the bounding boxes.

### Model details
-----------------
- The **input** is a batch of images of shape (m, 608, 608, 3)
- The **output** is a list of bounding boxes along with the recognized classes. Each bounding box is represented by 6 numbers `(p_c, b_x, b_y, b_h, b_w, c)` as explained above. If you expand `c` into an 80-dimensional vector, each bounding box is then represented by 85 numbers. 
- The YOLO architecture if 5 anchor boxes are used is: IMAGE (m, 608, 608, 3) -> DEEP CNN -> ENCODING (m, 19, 19, 5, 85)
  ![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/box_label.png?raw=true)
- Each cell gives you 5 boxes. In total, the model predicts: 19x19x5 = 1805 boxes just by looking once at the image (one forward pass through the network)! That is way too many boxes. Filter the algorithm's output down to a much smaller number of detected objects. 

### Filtering
-------------
To reduce the number of detected objects, apply two techniques:
1. Score-thresholding: 
  Throw away boxes that have detected a class with a score less than the threshold
  ![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/probability_extraction.png?raw=true)
2. Non-maximum suppression (NMS):
  ![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/non-max-suppression.png?raw=true)

  In this example, the model has predicted 3 cars, but it's actually 3 predictions of the same car. Running non-max suppression (NMS) will select only the most accurate (highest probabiliy) one of the 3 boxes.
  
 The steps to perform NMS are:
  1. Select the box that has the highest score.	  
  2. Compute its overlap with all other boxes, and remove boxes that overlap it more than iou_threshold.
  3. Iterate until there's no more boxes with a lower score than the current selected box.

For example, suppose there are two boxes, box1 and box2. 
p(box1) = 0.9
p(box2) = 0.6
iou(box1, box2) = 0

![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/iou.png?raw=true)


Steps followed:
1. Select box1 since it has the highest probability among the available scores.
2. Since there's no overlap, box1 is selected.
3. Iterating, box2 has the highest probability now. Select box2.
4. End of the loop.


### Results
-----------
Input image:
  ![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/0005.jpg?raw=true)

Output image:
  ![](https://github.com/adnaneaabbar/object-detection-for-autonomous-driving/blob/master/assets/images/0005_output.jpg?raw=true)

### NOTE
--------
Training a YOLO model takes a very long time and requires a fairly large dataset of labelled bounding boxes for a large range of target classes. This project uses existing pretrained weights from the official YOLO website, and further processed using a function written by Allan Zelener.

### References
--------------
1. Joseph Redmon, Santosh Divvala, Ross Girshick, Ali Farhadi - You Only Look Once: Unified, Real-Time Object Detection (2015)
2. Joseph Redmon, Ali Farhadi - YOLO9000: Better, Faster, Stronger (2016)
3. Allan Zelener - YAD2K: Yet Another Darknet 2 Keras
4. The official YOLO website (https://pjreddie.com/darknet/yolo/)
