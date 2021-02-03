---
published: true
title: Convolutional Neural Networks (CNN)- Step 1b- The Rectified Linear Unit (RELU)
date: 2018-12-28
layout: single
author_profile: false
read_time: true
tags: [machine learning , Deep Learning , CNN] 
categories: [deeplearning]
excerpt: " Deep Learning , CNN "
comments : true
toc: true
toc_sticky: true
---
----------

The Rectified Linear Unit, or ReLU, is not a separate component of the convolutional neural networks' process.

  

It's a supplementary step to the convolution operation that we covered in the previous tutorial. There are some instructors and authors who discuss both steps separately, but in our case, we're going to consider both of them to be components of the first step in our process.

  

If you're done with the previous section on artificial neural networks, then you should be familiar with the rectifier function that you see in the image below.

  

![](https://sds-platform-private.s3-us-east-2.amazonaws.com/uploads/71_blog_image_1.png)  

  

The purpose of applying the rectifier function is to increase the non-linearity in our images. The reason we want to do that is that images are naturally non-linear.

  

When you look at any image, you'll find it contains a lot of non-linear features (e.g. the transition between pixels, the borders, the colors, etc.).

  

The rectifier serves to break up the linearity even further in order to make up for the linearity that we might impose an image when we put it through the convolution operation. To see how that actually plays out, we can look at the following picture and see the changes that happen to it as it undergoes the convolution operation followed by rectification.

  

**The input image**

This black and white image is the original input image.

  

![](https://sds-platform-private.s3-us-east-2.amazonaws.com/uploads/71_blog_image_2.png)  

  

**Feature detector**

By putting the image through the convolution process, or in other words, by applying to it a feature detector, the result is what you see in the following image.

  

![](https://sds-platform-private.s3-us-east-2.amazonaws.com/uploads/71_blog_image_3.png)  

  

As you see, the entire image is now composed of pixels that vary from white to black with many shades of gray in between.

  

**Rectification**

What the [rectifier function](https://en.wikipedia.org/wiki/Rectifier_(neural_networks))  does to an image like this is remove all the black elements from it, keeping only those carrying a positive value (the grey and white colors).

  

The essential difference between the non-rectified version of the image and the rectified one is the progression of colors. If you look closely at the first one, you will find parts where a white streak is followed by a grey one and then a black one.

  

After we rectify the image, you will find the colors changing more abruptly. The gradual change is no longer there. That indicates that the linearity has been disposed of.

  

![](https://sds-platform-private.s3-us-east-2.amazonaws.com/uploads/71_blog_image_4.png)  

  

You have to bear in mind that the way by which we just examined this example only provides a basic non-technical understanding of the concept of rectification. The mathematical concepts behind the process are unnecessary here and would be pretty complex at this point.
