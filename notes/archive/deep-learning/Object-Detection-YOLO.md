---
title: "Object Detection"
date: 2019-05-19T19:40:23+10:00
draft: false
categories: ["Deep Learning"]
---

# Object Detection

Object detection deals with detecting instances of objects of a certain class, such as humans, animals, etc, in digital images and videos. Object detection has applications in many areas of computer vision, including image retrieval, face detection, video surveillance, and self-driving, etc. Current detection systems repurpose classifiers to perform detection. To detect an object, these systems take a classifier for that object and evaluate it at various locations and scales in a test image.

![](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Deep-Learning/YOLO.jpg)

Object detection has made important progress in recent years. Mainstream algorithms are divided into two types: (1) two-stage detectors, such as R-CNN algorithm. The main idea is to adopt the heuristic method (selective search) first, or regional proposal network (RPN) to generate a series of sparse candidate boxes, and then classifies and returns these candidate boxes. This kind of approach needs two shots to detect objects, one for generating region proposals, one for detecting the object of each proposal. Such models reach the highest accuracy rate, but are typically slower; (2) single-stage detectors, such as YOLO (You Only Look Once) and SSD (Single Shot MultiBox Detector), that treat object detection as a simple regression problem by taking an input image and learning the class probabilities and bounding box coordinates. The main idea is to uniformly sample at different positions of the image. Different scales and aspect ratios can be used for sampling. Then, using convolutional neural networks to extract features and directly classify and return, the whole process only needs a single step, so its advantage is Fast. But an important disadvantage of uniform sampling is that the training is difficult, mainly because the positive and negative samples are extremely unbalanced, resulting in slightly lower model accuracy.

![](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Deep-Learning/YOLOv3 Inference Time.png)

## YOLO

### Sliding Window

Image detection is harder than image recognition because, in image detection, there can be multiple objects, or even maybe even multiple objects of different categories within a single image. At this time, sliding windows can help. We define a window of some size and put the window over a region on the image. Then feeding the input region to the convolutional neural network model to get an output. Likewise, we repeat this process on each and every region of the image with a certain stride. Once done, we take a window of other sizes (longer, wider, etc) and slide the window over the image, and repeat this process again and again. We may probably end up with a window of the size of a snake in the image and seeing the model to output a breed for that window, meaning we detect a snake in that particular region.

### Bounding Box

One of the disadvantages of sliding windows is the computational cost. As we crop out many square regions in the image, we run each region on a model independently. We may think of using a bigger window as it will reduce computation but, as a cost, the accuracy will decrease dramatically. Bounding boxes are the boxes that enclose the object in an image. The idea of bounding boxes is to divide the image into grids and then for each grid we define our Y label with some arguments. P is the probability that there is an object in the grid cell. If P equals to 0, then the other arguments are all ignored. Bx, By, Bh, Bw are respectively the x coordinate, the y coordinate, the height, the width of the bounding box. C1, C2... Cn refers to the class probability that the object is of a specific class. The number of classes may vary, depending on whether it’s a Binary Classification or Multi-Class Classification. If a grid contains an object, i.e. P equals to 1, then we know there is an object in a certain region of the image. Now there are some issues we should consider, including how big is the size of the grid, which grid among all the girds whose P equals to 1 is responsible for outputting a bounding box for the object that span over multiple grids, etc. Usually, in practice 19 by 19 grid is used and the grid responsible for outputting the Bounding-Box for a particular object is the grid that contains the mid-point of the object. And, one more advantage of using 19 by 19 grid is that the chance of mid-point of the object appearing in two grid cells is smaller.

### Intersection Over Union

IOU means to divide the intersection of the bounding box and the true rectangle edge of the object by the union of them. The concept of intersection over union comes in to determine how accurate are these predictions. Suppose there are multiple bounding boxes an object in some grids, what intersection over union tells us is how close our prediction is to the ground truth. Then if the result is high enough (namely, greater than equal to a certain threshold) then the prediction is considered to be correct else we need to work on other bounding boxes.

![](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Deep-Learning/IoU.jpg)

### Non-Max Suppression

After doing intersection over union, for a prediction of a single object which spans over multiple grids, each grid would output its own prediction with a probability score, but it can make the predictions messy because there are multiple bounding boxes for a single object. What we do is, out of all the bounding boxes, we choose the box with the highest probability, and discard all the other boxes with a lower probability than the highest one. This is called non-max suppression.

### Anchor Box

The last problem is how to detect multiple objects in the same grid cell. It is easy to deal with. The idea is to define multiple bounding box prediction values, that is to have many probabilities, x and y coordinates, heights, widths, and class confidences in a single array to refer to the class probability of an object, and this is called anchor boxes. 
