---
layout: post
title: "deriving opengl perspective matrix from opencv intrinsic matrix"
description: ""
category: 
tags: [augmented reality, opencv, opengl, intrinsic matrix, camera calibration]
---
{% include/JB/setup %}

When we develop augmented reality applications, we have to display opengl graphics superimposed on the realtime video feed that you get from a camera.

To do this we must do two steps:
* We must first calibrate our camera as an offline process to determine the intrinsic parameters of the camera as described by [Hartley and Zisserman](http://users.rsise.anu.edu.au/hartley/public_html/Papers/CVPR99-tutorial/tutorial.pdf)
* The augmented reality application, on every frame of the realtime video feedback, now uses the intrinsic matrix, and correspondence between the image and object-centric points of a fiducial marker and give you the rotation and translation (model-view matrix) of the opengl frame.
For drawing an open opengl object, we need the current model-view matrix and the perspective matrix. We just mentioned how we can get the model-view matrix. The persective matrix conventionally stays the same for the duration of the application. How do we calculate the perspective matrix, given the intrinsic matrix we got from our camera calibration?



![My helpful screenshot]({{ site.url }}/assets/boulder.jpg)

Before we must give any explanation, we must acknowledge the following excellent blog post by <a title="Kyle Simek's explanation" href="//ksimek.github.io/2013/06/03/calibrated_cameras_in_opengl/">Kyle Simek </a> on this subject. What follows is my personal way to explain off the complexity, deriving largely from Kyle's work.

Let us start a look at the opengl perspective matrix. A point within the view frustum,
