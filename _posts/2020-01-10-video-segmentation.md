---
layout: post
title: "[Study] Video Segmentation"
categories: [general, setup, demo]
tags: [demo, dbyll, dbtek, setup]
fullview: true
comments: true
---
https://www.youtube.com/watch?v=0VH1Lim8gL8

## Video Segmentation

Model 종류
- FCN
- U-Net
- 3D Segmentation Model
- RNN + Segmentation Model (Sequence Video Segmentation)

### 3D Semantic Segmentation
1. An Efficient 3D CNN for Action/Object Segmentation on Video
- Unsupervised Learning
- Use 3D Seperable convolution(channel-wise(known as depth-wise) + point-wise)
- R(2+1)D Conv(Spatial + Temporal) : A Closer Look at Spatiotemporal Convolutions for Action Recognition
[ref site](https://blog.airlab.re.kr/2019/09/R(2+1)D)



Primary Metrics :
- IoU(Intersection over Union) or Jaccard Index
- F-measure(Contour Accuracy)


**dbyll** is brought to you by **[dbtek](http://ismaildemirbilek.com)**. Open sourced under [MIT](http://opensource.org/licenses/MIT) license.

<a class="btn btn-default" href="https://github.com/dbtek/dbyll">Grab your copy now!</a>
