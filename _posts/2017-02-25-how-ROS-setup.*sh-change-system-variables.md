---
layout: post
title:  "ROS setup.\*sh 如何改变系统环境变量？"
date:   2017-02-25
desc: "如果在命令行中执行'source setup.\*sh'，与ROS相关的系统环境变量的值就可能改变。"
keywords: "blog,ROS,shell,script,Linux"
categories: [ROS]
tags: [ROS,Linux]
icon: icon-html
---
已经知道，ROS命令行工具（例如rosrun）需要根据ROS\_PACKAGE\_PATH中指明的路径去各个workspace（工作空间）中搜索软件包，而每个工作空间的路径可以通过source相应的setup.\*sh加入到该环境变量中。看起来只要source所有的工作空间就对了，但是如果颠倒顺序是不行的，会出现写了source某个工作空间但环境变量中没有包含对应的路径。

那是不是要按照工作空间的第一次被catkin\_make的顺序去source呢？有时候可以，有时候也出问题，也不完全对。因为ROS\_PACKAGE\_PATH的值是被替换而不是附加。

其实，具体顺序根本就不是最关键的。根本上我们需要的只是source包含所有工作空间路径的setup.\*sh就行了。因为每一次catkin\_make某个工作空间都会生成新的setup.\*sh，而其中包含了当前的ROS\_PACKAGE\_PATH中的路径和当前的工作空间的路径。所以原则上，创建新工作空间后，在多个工作空间来回切换并catkin\_make，最后所有工作空间生成的setup.\*sh都是可以交替包括所有的工作空间路径的，但顺序略不同，这应该是这种机制的设计目的。

如果在\*shrc文件中写所有source工作空间的命令，ROS\_PACKAGE\_PATH中包的重叠结果由最后一条决定。所以最后一条最好是source最后catkin\_make的工作空间的setup.\*sh脚本。

工作空间之间最好不要有依赖关系。如果工作空间之间需要用ROS\_PACKAGE\_PATH指示依赖关系的话，千万不要在\*shrc文件中使用source指令，因为它会为新打开的终端设置固定的ROS\_PACKAGE\_PATH。应该使用在一个终端中使用source和catk\_make指令为一个工作空间管理自己的ROS\_PACKAGE\_PATH，防止同名包冲突，在其他终端中使用该空间时也source该空间自己的setup.\*sh。这种时候情况会变得复杂，需要具体分析。
