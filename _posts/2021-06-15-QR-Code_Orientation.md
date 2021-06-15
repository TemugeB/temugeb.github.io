---
layout: post
title:  "Orientation of QR code using OpenCV"
date:   2021-06-15 10:00:00 +0000
categories: Python Computer_vision
---

Determining the relative position of an object with respect to a camera viewport is useful in many applications. For example, AR applications need to define a coordinates system before applying augmentations. In our lab, we use QR code or AR markers to identify objects and determine their positions in our work space. We will use openCV in this post to check if a QR code exists in camera frame and if it does, determine the rotation matrix and translation vector from QR coordinate system to camera coordinate system. 

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/f000.gif?raw=true">
</p>
<p align="center">
Figure 1. Example output.
</p>

Before we start, you will need to have the intrinsic camera matrix and distortion parameters of your camera. To make this post more complete, a sample video clip and the correponding camera intrinsic parameters are provided. In this demo, the RGB channels of an Intel RealSense D435 camera is used. Some manufacturers will provide intrinsic camera parameters in product description. If not, you can follow my camera calibration guide for the single camera setup here: [link](https://temugeb.github.io/opencv/python/2021/02/02/stereo-camera-calibration-and-triangulation.html), or follow the OpenCV calibration guide here: [link](https://docs.opencv.org/master/dc/dbb/tutorial_py_calibration.html).

First, we add imports and simple code to read the camera parameters.
```python
import cv2 as cv
import numpy as np
import sys


def read_camera_parameters():

    inf = open('camera_parameters/intrinsic.dat', 'r')

    cmtx = []
    dist = []
    
    #ignore first line
    line = inf.readline()
    for _ in range(3):
        line = inf.readline().split()
        line = [float(en) for en in line]
        cmtx.append(line)
    
    #ignore line that says "distortion"
    line = inf.readline()
    line = inf.readline().split()
    line = [float(en) for en in line]
    dist.append(line)
    
    #cmtx = camera matrix, dist = distortion parameters
    return np.array(cmtx), np.array(dist)
```




