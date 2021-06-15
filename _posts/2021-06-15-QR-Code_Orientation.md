---
layout: post
title:  "Orientation of QR code using OpenCV"
date:   2021-06-15 10:00:00 +0000
categories: Python Computer_vision
---

Determining the relative position of an object with respect to a camera viewport is useful in many applications. For example, AR applications need to define a coordinates system before applying augmentations. In our lab, we use QR code or AR markers to identify objects and determine their positions in our work space. I will use openCV in this post to check if a QR code exists in camera frame and if it does, determine the rotation matrix and translation vector from QR coordinate system to camera coordinate system. 

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/f000.gif?raw=true">
</p>
<p align="center">
Example output
</p>
