---
layout: post
title:  "Aligning depth image to color frame"
date:   2023-09-28 10:00:00 +0000
categories: Python LibRealSense
---

In this post, I give an overview of how to align a depth image to a color image frame. The motivation was that my Realsense camera was connected to a Pi, which did not have the processing power to align depth to color. This meant I needed to send the images to a workstation PC, which can then do the alignment quickly.
This blog post accompanies the demo code uploaded here: [github](https://github.com/TemugeB/manual_align_depth_to_color). Running the demo code first may be helpful.

Before we start, I should clarify that this post shows how to align depth image to color frame by manually manipulating projection matrices. If you already have a Realsense camera and just want to use their provided libaries, go here: [github](https://github.com/IntelRealSense/librealsense/blob/master/wrappers/python/examples/align-depth2color.py). 
Additionally, if your Realsense camera is not physically connected to your PC, Intel's suggested solution is to turn your camera into a network device, which then becomes a physically connected device through your LAN. Here is their solution: [link](https://dev.intelrealsense.com/docs/open-source-ethernet-networking-for-intel-realsense-depth-cameras). So you can still use Realsense SDK on a remote workstation. Also, if you're using a more powerful edge device (i.e Nvidia Jetson etc), you can build the Realsense SDK with CUDA support, which will then have no problem running the alignment.

**Alignment workflow**

To align the depth image, the workflow will be:
1. Convert depth image to pointcloud in depth camera coordinates.
2. Convert pointcloud in depth camera coordinates to color camera coordinates.
3. Project pointcloud in color camera coordinates to color camera frame pixels.

**1. Convert depth image to pointcloud in depth camera coordinates**

The figure below shows how the depth image is setup. We are looking at only the x axis pixels in this example, but it is the same for y axis.

![image](https://github.com/TemugeB/temugeb.github.io/assets/36071915/4345d961-ed2c-41db-a4f2-9a87f540c121)
Here, _d_ is the depth value of the pixel _px_ and _fx_ is the focal point of the camera. So the x position of the point in 3D space can be calculated from similar triangles as:

![image](https://github.com/TemugeB/temugeb.github.io/assets/36071915/cecbc2ca-6b4a-4f45-bb5d-6b5dc76f42f8)

We see here that _d_ is dependent on the image but _fx_ and _px_ are depened on the camera settings. So the constants in the parenthesis can be precalculated. Once we get a depth image, we simply multiple each depth pixel by a constant to get the x and y coordinates in 3D space to make a pointcloud. In the demo code, this is done so:

```python

#precalculate values needed to convert depth image to pointcloud
def prep_XY():

    #focal point and center of frame
    height = depth_intrinsics.height
    width = depth_intrinsics.width
    ppx = depth_intrinsics.ppx
    ppy = depth_intrinsics.ppy
    fx = depth_intrinsics.fx
    fy = depth_intrinsics.fy

    #create grid for point_cloud calculation
    points_xy = np.mgrid[0:height, 0:width].reshape((2, -1)).T

    #make the grid points centered
    points_xy = points_xy - np.array([ppy, ppx])
    points_x = points_xy[:,1]
    points_y = points_xy[:,0]

    #scaled_points
    points_x = points_x/fx
    points_y = points_y/fy

    return points_x, points_y
```
This is an expensive calculation that should not be in your main loop. It needs to be prepared only once and reused. So once we have a depth image, we can get the pointcloud simply by direct multiplication:

```python

#this code converts depth image to pointcloud.
def depth_to_pointcloud(depth: np.ndarray, points_x: np.ndarray, points_y:np.ndarray) -> np.ndarray:

    Z = depth.flatten()
    #calculate the x,y coordinates of the point cloud
    X = (points_x * Z)
    Y = (points_y * Z)

    #This is the pointcloud
    pcd = np.stack([X,Y,Z], axis = -1)

    return pcd
```

**2. Convert pointcloud in depth camera coordinates to color camera coordinates.**

Now the pointcloud is in depth camera coordinates system. To align with the color camera frame, we first need to transform the pointcloud to be in color camera coordinate system. This is done easily with simple matrix multiplication:

```python

    #convert the depth image to pointclound in depth camera coordinates
    pcd = depth_to_pointcloud(depth, points_x, points_y)

    #pointcloud in depth camera coordinates to pointcloud in color camera coordinates
    rot = np.array(depth_to_color_extrinsics.rotation).reshape((3,3))
    trans = np.array(depth_to_color_extrinsics.translation).reshape((1,3))
    #translation vector is in units of [meter] while pointcloud is in [milimeter]. So a convertion is necessary
    trans *= 1000.
    pcd_color = pcd @ rot + trans
```
The last line converts pointcloud from depth to color coordinate system. However, the extrinsic transformation needs to be known from somewhere.


**3. Project pointcloud in color camera coordinates to color camera frame pixels.**

So now the pointcloud is in color camera coordinate system. We simply project the pointcloud points using the color camera intrinsic parameters. We do have to be careful about the holes in the original depth image.
Projection is done using the pinhole camera model. The exact equation and explanation can be found here: [openCV](https://docs.opencv.org/3.4/d9/d0c/group__calib3d.html#:~:text=equivalent%20to%20the%20following)

```python

    height = color_intrinsics.height
    width = color_intrinsics.width
    ppx = color_intrinsics.ppx
    ppy = color_intrinsics.ppy
    fx = color_intrinsics.fx
    fy = color_intrinsics.fy

    #split the pointcloud into its coordinates along each axis
    Xc = pcd_color[:,0]
    Yc = pcd_color[:,1]
    Zc = pcd_color[:,2]

    #these are indices of depth points that are valid. 
    nonzeros = Zc != 0

    #project to pixel coordinates. Only pointcloud points that are valid.
    u = fx * (Xc[nonzeros]/Zc[nonzeros]) + ppx
    v = fy * (Yc[nonzeros]/Zc[nonzeros]) + ppy
    #convert to image like indexing
    u = np.round(u).astype(np.int32)
    v = np.round(v).astype(np.int32)
```
Now _u,v_ contains the pixel coordinates of the pointcloud in color camera space. However, some pixels may fall outside the color camera frame. We simply remove these pixels and fill the new aligned depth image with the corresponding depth values.

```python

    #Now we are only interested in points that fall inside the color image frame
    u_inbound = (u >=0) * (u < width)
    v_inbound = (v >=0) * (v < height)
    inbound = u_inbound * v_inbound #these are indices of points that are inside the color camera frame
    
    #shortlist the pointcloud to only non-zero and in frame points
    u = u[inbound]
    v = v[inbound]
    z = Zc[nonzeros][inbound] #these are the depth values for each pixel

    #create a new frame for the aligned depth image
    manual_align = np.zeros((height, width), dtype = np.uint16)
    #apply the pixel values
    manual_align[(v, u)] = z
```

That is it. Here, _manual_align_ is a depth image that has been aligned with the color camera frame. However, there are some inaccuracies which I will discuss below.

**Issues**

When calculating the _x,y_ coordinates of the pointcloud, we assumed the depth pixels were centered, which they are not. In other words, we are only converting the top left corner of the depth pixels to pointcloud. A correct implementation should center the pixels before converting to pointcloud.
Additionally, sometimes we will see lines of emtpy values in the aligned depth images. This is because we round the projected pixel values to the closest integer. Some of these lines will disappear if we map both top left and bottom right corner of depth pixel to pointcloud then to color image. But this is expensive. A cheating way to get rid of these lines is to simply use a small median filter:
```python
depth = cv.medianBlur(depth, 3)
```
