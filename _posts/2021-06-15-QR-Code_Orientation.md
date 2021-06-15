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

First, we add imports and simple code to read the camera parameters. If you're using your own intrinsic parameters, make sure to follow the format in my example or rewrite this code. 
```python
import cv2 as cv
import numpy as np
import sys


def read_camera_parameters(filepath = 'camera_parameters/intrinsic.dat'):

    inf = open(filepath, 'r')

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

if __name__ == '__main__':

    cmtx, dist = read_camera_parameters()
```

Next, we will define input stream source. If you want to use a webcam, simply call the code with webcam id from command line. Otherwise, the default behavior is to use provided sample video clip. 
```python
if __name__ == '__main__':

    #read camera intrinsic parameters.
    cmtx, dist = read_camera_parameters()

    input_source = 'test.mp4'
    if len(sys.argv) > 1:
        input_source = int(sys.argv[1])

    show_axes(cmtx, dist, input_source)
```

Next, we will create a QR code reader object. Additionally, we will open the input stream source and read frames one by one and show the output.
```python
def show_axes(cmtx, dist, in_source):

    cap = cv.VideoCapture(in_source)

    qr = cv.QRCodeDetector()

    while True:
        ret, img = cap.read()
        if ret == False: break

        cv.imshow('frame', img)

        k = cv.waitKey(20)
        if k == 27: break #27 is ESC key.

    cap.release()
    cv.destroyAllWindows()
```

Next, we run the QR detection algorithm. This can be done with a single call.
```python
ret_qr, points = qr.detect(img)
```
The first returned value indicates if QR code was found or not, and has a boolean value. The second returned variable provides the four corners of the QR code. These are shown in Figure 2. If you want to detect QR and decode the value, you can call detectAndDecode(). 


<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/QR_points.png?raw=true">
</p>
<p align="center">
Figure 2. Points returned by QR detector.
</p>

OpenCV uses a right hand coordinate system. To have the QR coordinate axes to point up, we have to chose x axis to be pointing from point #1 to #4 and y axis to point from #1 to #2. This will point z axis up from the QR code. These coordinates need to be saved to draw them later. I add these coordinates at the top of my code as global variables. 
```python
import cv2 as cv
import numpy as np
import sys

qr_edges = np.array([[0,0,0],
                     [0,1,0],
                     [1,1,0],
                     [1,0,0]], dtype = 'float32').reshape((4,1,3))
...
```

