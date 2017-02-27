---
layout: post
title:  "ROS的setup脚本如何改变系统环境变量？"
date:   2017-02-25
desc: "如果在命令行中执行setup脚本，与ROS相关的系统环境变量的值就可能改变。"
keywords: "blog,ROS,shell,script,Linux"
categories: [Ros]
tags: [ROS,Linux]
icon: icon-html
---

[TOC]
## 测试的环境

- ROS版本：indigo

- 操作系统：Ubuntu 1404 32bit/64bit

## 准备知识

完整地学习过[ROS beginner level](http://wiki.ros.org/ROS/Tutorials#Beginner_Level)。

## 问题

我们已经知道，某些ROS命令行工具（例如rosrun）需要根据这个环境变量，ROS\_CKAGE\_PATH，指明的路径去搜索某些ROS软件包。同时，我们也知道可以通过source某个ROS工作空间（workspace）相应的setup.\*sh使该工作空间的src文件夹路径在该环境变量中。

但是，一个setup.\*sh做了哪些事，又是如何完成自己的工作的呢？



## 探究

一个工作空间中的所有setup文件（setup.\*sh）最后都指向setup.sh。在setup.sh中，有这样的一段话：

```


#!/usr/bin/env sh

# generated from catkin/cmake/template/setup.sh.in



# Sets various environment variables and sources additional environment hooks.

# It tries it's best to undo changes from a previously sourced setup file before.

# Supported command line options:

# --extend: skips the undoing of changes from a previously sourced setup file



# since this file is sourced either use the provided _CATKIN_SETUP_DIR

# or fall back to the destination set at configure time

```

翻译过来就是：

```

......

设置很多环境变量和sources某些额外的环境钩子。

它尽自己最大的能力来撤销先前被source的setup文件的影响。

一命令行选项被支持：

--extended：跳过对来自先前被source的setup文件的修改的撤销。

......

```

在除'--extend'选项外的默认情况下，一个setup.\*sh会尽可能撤销先前其他所有setup.\*sh的影响，然后它再自己施加影响。

## 应用

### 单工作空间（workspace）

如果我们只创建一个工作空间A且只在它里面工作。为了向一些ROS有关的环境变量中加入该工作空间有关的值，我们可以有“省事”和“不省事”两种做法。

#### “不省事”的做法

在每次打开一个shell（Linux中的命令行）时都source对应的setup文件。bash shell对应`A/devel/setup.bash`，zsh shell对应`A/devel/setup.zsh`。如下：

```

source A/devel/setup.*sh

```

#### “省事”的做法

 为了省事，我们也可以把这句命令写入到该shell相应的rc文件（bash shell对应的.bashrc或者zsh shell对应的.zshrc）中的适当位置处（至少应该放在`source /opt/ros/indigo/setup.*sh`后）。



这两种做法都没问题，只是第一种稍微麻烦。

### 多工作空间 （workspace）

在`/opt/ros/indigo/`外，如果我们创建了多个工作空间且平常在多个工作空间中来回地工作，那么情况会是怎么样？比如我有两工作空间：

1. /home/kun/catkin_ws

2. /home/kun/catkin_test



怎样做才能满足我们的需求呢？想这个问题时，不免想到情况是什么？我们的目的或者需求是什么？情况是ROS\_PACKAGE\_PATH变量中的**多个目录是有先后顺序的**。目的是我们的**目的各不相同**。所以我把做法按照目的分成两大类。

#### “不省事”的做法

每次打开一个shell时，按照我们要去编写、编译或运行的软件包的依赖关系，依次source相应的工作空间的setup文件。比如：工作空间2中的软件包A依赖工作空间1中的一些软件包。那么可以执行：

```

source /home/kun/catkin_ws/devel/setup.*sh --extended

source /home/kun/catkin_test/devel/setup.*sh --extended

```

如果当前包没有依赖所有其他工作空间的某些包，则只需在该shell中source当前工作空间的那个setup文件。

```

source /home/kun/catkin_test/devel/setup.*sh

```

#### “省事”的做法

##### 情况1

如果工作空间1中所有软件包和工作空间2中的所有软件包之间没有任何依赖关系，那么，将下面两句话放在该shell相应的那rc文件中合适的位置（至少应该放在`source /opt/ros/indigo/setup.*sh`后）。


```

source /home/kun/catkin_ws/devel/setup.*sh --extended

source /home/kun/catkin_test/devel/setup.*sh --extended

```

因此，变量ROS\_PACKAGE\_PATH形成一个顺序值

```

ROS_PACKAGE_PATH=/home/kun/catkin_test/src:/home/kun/catkin_ws/src:/opt/ros/indigo/share:/opt/ros/indigo/stacks

```

**但注意：**此时，如果/home/kun/catkin_ws/和opt/ros/indigo/share/中有一对同名软件包，/home/kun/catkin_ws/中的会遮蔽opt/ros/indigo/share/的。

比如，我在opt/ros/indigo/share/中安装了image_proc包，又在/home/kun/catkin_ws/中通过下载源代码的方式安装了这个包。而我在编译某项目时想使用opt/ros/indigo/share/中的这个包，但是由于ROS_PACKAGE_PATH环境变量中多个目录的固定的顺序，我却会使用到/home/kun/catkin_ws/中的那个包。以rospack命令演示一下，对于该包，总有且只有一个路径会被先找到。

```

➜  ~ rospack find image_proc

/home/kun/catkin_ws/src/image_pipeline/image_proc

```

##### 情况2

如果只有工作空间2中某些软件包依赖工作空间1中的某些软件包，那么，将下面两句话放在该shell相应的那rc文件中合适的位置（至少应该放在`source /opt/ros/indigo/setup.*sh`后）。

```

source /home/kun/catkin_ws/devel/setup.*sh --extended

source /home/kun/catkin_test/devel/setup.*sh --extended

```


或者，如果只有工作空间1中某些软件包依赖工作空间2中的某些软件包。那么，将下面两句话放在该shell相应的那rc文件中合适的位置（至少应该放在`source /opt/ros/indigo/setup.*sh`后）。

```

source /home/kun/catkin_test/devel/setup.*sh --extended

source /home/kun/catkin_ws/devel/setup.*sh --extended

```

##### 情况3

如果工作空间1中某些软件包与工作空间2中的某些软件包相互依赖，这是最复杂的交叉依赖情况。

请**尽量不要这么做**，因为这是在给自己找麻烦。

如果你不得不这样做，那么请不要像前面那些“省事”的方法那样修改某rc文件。**请直接使用前面的“不省事”的做法**，因为那就是最省事的方法。
