---
layout: post
title:  "Estimating joint angles from 3D body poses"
date:   2021-09-16 10:00:00 +0000
categories: Python Motion_capture 
---

This post gives a general strategy on how to calculate joint angles from 3D body poses given in world coordinates. If you need to get 3D body poses, check my post [here](https://temugeb.github.io/python/computer_vision/2021/09/14/bodypose3d.html). 

There are three main points on how to calculate the joint angles.  
1. Obtaining a rotation matrix that rotates one vector to another.  
2. Decomposing rotation matrix into rotation sequence along major axes. For example ZXY or XYZ rotation orders.
3. Setting up basic body pose. Also known as T pose.

**1. Obtaining a rotation matrix that rotates one vector to another.**

We can rotate a general vector **A** into **B** by defining a normal direction given **A**x**B** and then rotating along this new axis.  

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/rots.png?raw=true" height = 300>
</p>
