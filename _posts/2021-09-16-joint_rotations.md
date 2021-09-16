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

We can rotate a general vector **A** into **B** by defining a normal direction given **A**x**B** and then rotating along this new axis, as shown in Fig. 1. Our strategy is then to change coorindates to the one defined by **A**, **B**, **A**x**B**, rotate in this basis and then rotate back to original basis.   

<p align="center">
  <img src="https://github.com/TemugeB/temugeb.github.io/blob/main/_posts/images/rots.png?raw=true" height = 280>
</p>
<p align="center">
Figure 1. Rotation along normal direction.
</p>


Given vector **A** and **B**, we can define a new basis by making **A** a unit vector and taking the rejection of **B** onto **A** as the second axis. The third axis is simply defined as **A**x**B** normalized. This will allow us to write down the change of basis rotation matrix. Once the coordinates are aligned, we can simply rotate along the **A**x**B** direction, which is given by Rz rotation matrix. Note however that we don't need to know the rotation angle, since cos(theta)
and sin(theta) are simply dot and cross products of the normalized **A** and **B** vectors.
```python 

#calculate rotation matrix to take A vector to B vector
def Get_R(A,B):

    #get unit vectors
    uA = A/np.sqrt(np.sum(np.square(A)))
    uB = B/np.sqrt(np.sum(np.square(B)))

    #get products
    dotprod = np.sum(uA * uB)
    crossprod = np.sqrt(np.sum(np.square(np.cross(uA,uB)))) #magnitude

    #get new unit vectors
    u = uA
    v = uB - dotprod*uA
    v = v/np.sqrt(np.sum(np.square(v)))
    w = np.cross(uA, uB)
    w = w/np.sqrt(np.sum(np.square(w)))

    #get change of basis matrix
    C = np.array([u, v, w])

    #get rotation matrix in new basis
    R_uvw = np.array([[dotprod, -crossprod, 0],
                      [crossprod, dotprod, 0],
                      [0, 0, 1]])

    #full rotation matrix
    R = C.T @ R_uvw @ C
    #print(R)
    return R

```
