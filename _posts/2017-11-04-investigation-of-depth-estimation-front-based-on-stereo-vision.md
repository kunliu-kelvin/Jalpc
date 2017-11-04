---
layout: post
title:  "基于立体视觉的深度估计的前沿调研"
date:   2017-11-04
desc: "基于立体视觉的深度恢复是个很重要的问题，近年来，深度学习为这个问题带来了新的解决思路。"
keywords: "Machine Learing,Depth Recovery"
categories: [Machine_learning]
tags: [Depth Recovery, Machine Learning, Robot Vision]
icon: icon-html
---

* TOC
{:toc}

## 基准测试和挑战赛
[The KITTI Vision Benchmark Suite](http://www.cvlibs.net/datasets/kitti)(A project of Karlsruhe Institute of Technology and Toyota Technological Institute at Chicago)中有立体视觉方面的基准测试（benchmarks）：Stereo Evaluation 2012 和 Stereo Evaluation 2015。[Stereo Evaluation 2015](http://www.cvlibs.net/datasets/kitti/eval_scene_flow.php?benchmark=stereo)是最新的基准测试。另一方面，[Stereo Evaluation 2012 - The KITTI Vision Benchmark Suite]()使用了一些容易被理解的性能指标：
- Out-Noc: Percentage of erroneous pixels in non-occluded areas
- Out-All: Percentage of erroneous pixels in total
- Avg-Noc: Average disparity / end-point error in non-occluded areas
- Avg-All: Average disparity / end-point error in total
- Density: Percentage of pixels for which ground truth has been provided by the method

有很多论文和很多方法使用了这个基准测试，其中不少开放了源代码。有些方法是基于深度学习的模型的，有些不是。测试时，使用195对图像，运行时间从0.03秒到300多秒。重建离差图的典型效果如下：

 > 按Out-Noc排名时，前三个结果

在该页面上，还有一些其他的相关的基准测试的数据集。看以后的需要，可以再拿来用。

## 基于深度学习的离差图（Disparity）估计的思路创新
Alex Kendall 是剑桥大学的研究院，研究i计算机视觉和机器人学。他提出了利用基本几何的深度学习框架 GC-Net 来估计离差图。
> We proposed the architecture GC-Net which instead looks at the problem’s fundamental geometry. see [Have We Forgotten about Geometry in Computer Vision? - Home](http://alexgkendall.com/computer_vision/have_we_forgotten_about_geometry_in_computer_vision/)

此刻，GC-Net 在 Stereo Evaluation 2012 中排名第二，在 Stereo Evaluation 2015 中排名第四。
