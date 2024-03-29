# 真机实验搭建过程


## 实验场景准备

我们项目的任务是控制一个机器人将桌面上的箱子抓取，然后摆放到一个容器区域内，所以需要安排一个初始堆放区域和一个目标装箱区域，需要两个相机分别拍照这两个区域。最终我们的布局如下：

![scene](./images/scene.png)

由于机器臂高度有限，为了更大的可操作空间，我们找了两个比较矮的桌子，平面高度都低于机器臂高度，确保我们划定的区域机器人都能够得着。下面介绍一下硬件配置。


### 机器人配置
使用的是UR5机器臂，完整的机器人是来自蓝胖子公司研发第一款复合协作机器人MOMA。

它由可移动的底座车（目前已不可操控）和UR5机器臂组合而成。中间由一个定制的钢直管（大概这意思）连接。这个机器人的连接方式存在一个问题：晃动。机器臂移动幅度较大的时候，底座容易跟着晃动，这也为后续的误差带来一定的影响。

我们抓取箱子用的端拾器是Robotiq的epick吸盘手，它内置了真空发生器，不用外接一个空压机什么的，挺好，就是吸取的时候声音比较吵闹。但这类吸盘吸取后如果还是很吵，一般是因为存在漏气现象，如果吸取的是手机盒子这种表面平滑的箱子，它吸取后内部的气压比较稳定，这时候不会继续抽气维持气压差，就不会吵了，所以为了安静实验也可以考虑用高质量纸盒子。

---
<details>
<summary>
有关吸盘的介绍与使用
</summary>

xxx
</details>

---

### 相机配置
使用的是两台微软的Azure KinectDK。

这款深度相机成像质量挺高，深度信息的噪点也比较少（对比老版的KinectV2来说），有着功能丰富的驱动软件，还可以当正常相机进行录像。可能是红外成像的通病，这个相机拍摄的点云在竖直平面上多少有些曲折变形，物体边缘也会有些点云和背景更加接近。比如下面的例子：


当物体背后有个大墙面的时候，这种变形的现象会更加明显。暂不知道有什么应对方法。

---
<details>
<summary>
关于Kinect相机的说明
</summary>

TODO
</details>

---

### 电脑配置

我们的系统最终用到了3台电脑：机器人电脑、相机电脑、算法电脑。

控制机器臂的电脑系统为ubuntu14，记得配有显卡，还是能跑一下神经网络的，没尝试过，主要还是在上面用ROS控制机器臂运动。

还有一台笔记本，系统是ubuntu18，这台笔记本是为了连接kinect相机的，因为这个新相机的驱动没法在ubuntu14上安装，就没办法让机器人的电脑直接连接相机，这意味着我们要在笔记本上连接相机，通过上面的ROS获取图像后，发送给机器人电脑的ROS节点。

好在ROS支持多设备的节点通讯，其中一台作为roscore的执行机器，接收管理其他电脑的节点信息，这里我们把机器人电脑作为主节点，将笔记本ROS的IP改成机器人电脑的IP，这样笔记本上执行的节点会直接通讯到机器人上，这种方式也支持不同版本的ROS通讯。这带来的好处是低版本ROS的机器人可以用高版本ROS才有的库的算法，这个在后面视觉标定会体现。

---
<details>
<summary>
如何跨电脑进行ROS通讯
</summary>

xxx
</details>

---

但跨电脑有个明显的坏处是更大的通讯代价，由于网线不够长，也没去弄路由器，笔记本我们是wifi连接校园网的，机器人是直连网线，两者的数据传递本来就不算快，我们还需要将kinect的RGBD数据发送给机器人电脑，中间的延迟更加大，最终我感觉从相机拍照到机器人电脑上接收图像可能有2s以上的延迟。所以这种情况还是别遇到的好。

最终还有远方的服务器电脑，这个是用来跑系统中各种神经网络的，机器人电脑获取各种信息后发送给服务器，服务器计算后返回给机器人去规划执行。这里又存在延迟，特别是一张完整的1080p的RGBD图片发送给服务器是很费时间的。我们没有在服务器上通过ROS传递信息了，而是socket通讯，这个后面用了一些方法提速了一下。但整体系统的响应还是比较慢的。


## 系统前置准备

在编写我们系统的逻辑之前，需要先保证各个模块的代码可用。

### 机器人系统
主要是基于UR官方的ROS代码改的，实现了Robot类，可以进行路径规划和运动，也可以避障规划，以及控制吸盘手开启关闭。


### 视觉系统


## 具体算法细节

### 多设备通讯与延迟


### 视觉检测



## 道具组



## 关于误差