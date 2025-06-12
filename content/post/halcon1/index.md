---
title: "Halcon基础"
description: 
slug: "halcon1"
date: 2025-06-12
categories:
    - 视觉
---

## 基础知识

------

光学：几何光学，物理光学

数学：导数为主的高等数学，矩阵论

## 五种需求：

------

1.识别定位

2.符号识别：一二维码，OCR

3.测量需求

4.缺陷需求（最常见，难度最大）

5.手眼标定和抓取（结合运动控制）

## 图像处理一般思路

------

1.采集

2.预处理

- 拉开灰度
- 几何变换
- 去噪：中值滤波，均值滤波，高斯滤波
- 抠图

3.图像分割

- 二值化
- 形态学
- 特征选择

ps：Halcon里区域和图像是不同概念

4.识别显示

5.通信

## 三大数据类型

------

图像，区域，XLD

## 灰度直方图

------

- 勾选“阈值”

  将灰度值在”绿线和红线之间”的以选定颜色进行填充
  ![img](https://s2.loli.net/2025/06/12/UCZYsDGtubjW6n7.png)

- 勾选“缩放”

  将把圈定的阈值范围内的直方图均匀拉伸释放到整个直方图轴上
  ![img](https://s2.loli.net/2025/06/12/F2nKR39vsMkCZiu.png)
  ![img](https://s2.loli.net/2025/06/12/PMOJhvNQ21KCeRd.png)

## 数组语法

------

```
* Simple tuple operations

Tuple1 := [1,2,3,4,5]
Number := |Tuple1|
SingleElement := Tuple1[3]
Part := Tuple1[1:3]
Copy := Tuple1[0:|Tuple1| - 1]
* Simple tuple operations

Tuple1 := [1,2,3,4,5]
Number := |Tuple1|
SingleElement := Tuple1[3]
Part := Tuple1[1:3]
Copy := Tuple1[0:|Tuple1| - 1]
```

运行结果
![img](https://s2.loli.net/2025/06/12/8Mx13QkwSV7s5uZ.png)

## 读取图片的四种方法

------

1. `文件 -> 读取图片`

2. `Image Acquisition -> 自动检测接口（刷新设备）-> Direct show`，从摄像头直接读图

3. `Image Acquisition -> 选择文件`，从图像文件中读取

4. `Image Acquisition -> 选择路径`，结合正则表达式读取路径下的图片

   PS：用Image Acquisition读取时记得点击代码生成
   ![img](https://s2.loli.net/2025/06/12/CLGzTRN4Xom6y8k.png)

摄像头抓取模式：在`可视化 -> 更新窗口`中调整

同步采集：实时抓取，一直抓取

异步采集：只等图片处理完后，grab_image才开始抓取

PS：更多信息包括双相机采集，可以在`案例 -> 方法 -> 图像采集设备`中学习

## ROI（感兴趣区域）

------

![img](https://s2.loli.net/2025/06/12/YxKfIQpq7ir9DFs.png)

## 特征检测

------

![img](https://s2.loli.net/2025/06/12/jKMsyDRJ5qliunT.png)

PS：二值化之后的区域虽然不连通，但仍然认为是一个区域。需要调用connection把区域分割成独立区域，才能进行select_shape，点击特定区域，再进行特征检测，则显示特定区域的特征信息。（connection分割依据（九宫格中的八邻域））

若直方图窗口内的直方图不显示

1.将程序执行一遍（按完全执行或F5）

2.将直方图窗口纵向拉大（这个拉升方案默认把剩余显示区域留给直方图）

## 形态学

------

膨胀：

- 将模板在原图上移动，当除中心点外的模板点，与原图中像素重合时，将中心点变黑。最终结果（当然也能覆盖原图）就是新图
- 效果即扩大了“一圈”

腐蚀：

- 将模板在原图上移动，仅有整个结构元素能全部被区域所包含时，保留中心元素点，最终结果即新图能被原图包含
- 效果即缩小了“一圈”

开运算：

- 先腐蚀，后膨胀
- 效果：减少像素（弱于腐蚀），断开

闭运算：

- 先膨胀，后腐蚀
- 效果：增加像素（弱于膨胀），连接

PS：以上是对二值图像而言，而对灰度图像做形态学处理是改变亮暗

## 代码运行事件监测

------

1. 使用count函数
2. 如图使用“性能评测器”

![img](https://s2.loli.net/2025/06/12/TezxKvo8ksC7wXH.png)

## 结构元素模板选择

------

- 要根据感兴趣的图像结构
- 来灵活选择结构元素模板

效果展示(以官方例程ball.hdve为例)：

- 原图
  ![img](https://s2.loli.net/2025/06/12/XeOxK4ZS28Y6GdE.png)
- 红色->圆结构元素提取
  ![img](https://s2.loli.net/2025/06/12/UkJFof7byRYqAiQ.png)
- 绿色->矩形结构提取
  ![img](https://s2.loli.net/2025/06/12/g4w81aiMmKGZ2Ff.png)

可以知道，想要提取圆的时候，采用圆结构元素能更好保留形态

之后采用圆度特征进行进一步的blob提取

## 预处理图像增强

------

一般通过如下几种方式：

1. scale_image: g’ := g * Mult + Add

   g为当前的灰度值，Mult 为所乘的系数，Add为加的偏移值，由公式可以刊出用scale_image来处理图像是个线性变化，会让黑的地方更黑，亮的地方更亮。

2. 图像形态学

   - gray_opening

     结构元素在图像中滑，灰度值最高的值作为新值，有使图像变亮的作用。

   - gray_closing

     结构元素在图像中滑，灰度值最低的值作为新值，有使图像变暗的作用。

   - gray_range_rect

     用一个矩形结构元素在图像中滑动，新值=(矩形中最大的)灰度值-(矩形中最大的)最小的灰度值

   - emphasize: res := round((orig - mean) * Factor) + orig

     mean代表先对原图进行mean_image后的图像对应的灰度值，

     orig 代表每幅图对应的灰阶值 ，res代表输出图像的灰阶值

## 凸性

------

- 凸性：图形内任意两点相连，图像上所有的点相连的的线进行点填充

```
函数：shape_trans(Region, RegionTrans, 'convex')
```

- 外接矩形：不赘述

```
函数：shape_trans(Region, RegionTrans, 'rectangle1')
```

PS：rectangle1为平行于窗口的正矩形，rectangle2为斜外接矩形，详见官方手册

以下图为例，凸形和外接矩形分别如下

![img](https://s2.loli.net/2025/06/12/v4Neal3FsBbHow6.png) ![img](https://s2.loli.net/2025/06/12/tn5Olj7T8U9zD2Q.png) ![img](https://s2.loli.net/2025/06/12/wTlHv4dY5PiLAgn.png)

## 定位方法

------

1. Blob分析
2. 模板匹配
   - 通过mark点或特征找出兴趣区域
   - 仿射变换到标准位置

## 仿射变换

------

以官方例程check_blister.hdev介绍

![img](https://s2.loli.net/2025/06/12/Qn3Xu2B5vPoMOR8.png) ![img](https://s2.loli.net/2025/06/12/wlnkZrcIdt4O7yx.png) ![img](https://s2.loli.net/2025/06/12/s2GovKnrytRgSVe.png)

```
dev_update_window ('off') //停止更新窗体
dev_close_window () //关闭窗体
read_image (ImageOrig, 'blister/blister_reference')
dev_open_window_fit_image (ImageOrig, 0, 0, -1, -1, WindowHandle) //窗口大小适应图片
access_channel (ImageOrig, Image1, 1) //通道1，R图
threshold (Image1, Region, 90, 255) //二值化，灰度直方图工具
shape_trans (Region, Blister, 'convex') //凸包，内部都被填充
orientation_region (Blister, Phi) //区域方向，获得角度值Phi，范围[-Π,Π)
area_center (Blister, Area1, Row, Column) //获得面积、质心坐标
vector_angle_to_rigid (Row, Column, Phi, Row, Column, 0, HomMat2D) // 获得仿射变换矩阵HomMat2D
affine_trans_image (ImageOrig, Image2, HomMat2D, 'constant', 'false') //对图像进行仿射变换，插值算法constant
dev_update_window ('on') //更新窗体
dev_update_window ('off') //停止更新窗体
dev_close_window () //关闭窗体
read_image (ImageOrig, 'blister/blister_reference')
dev_open_window_fit_image (ImageOrig, 0, 0, -1, -1, WindowHandle) //窗口大小适应图片
access_channel (ImageOrig, Image1, 1) //通道1，R图
threshold (Image1, Region, 90, 255) //二值化，灰度直方图工具
shape_trans (Region, Blister, 'convex') //凸包，内部都被填充
orientation_region (Blister, Phi) //区域方向，获得角度值Phi，范围[-Π,Π)
area_center (Blister, Area1, Row, Column) //获得面积、质心坐标
vector_angle_to_rigid (Row, Column, Phi, Row, Column, 0, HomMat2D) // 获得仿射变换矩阵HomMat2D
affine_trans_image (ImageOrig, Image2, HomMat2D, 'constant', 'false') //对图像进行仿射变换，插值算法constant
dev_update_window ('on') //更新窗体
```

## 测量助手

------

打开方法： `助手 -> 打开新的Measure`

![img](https://s2.loli.net/2025/06/12/lXqBbeMdak6ouKH.png) ![img](https://s2.loli.net/2025/06/12/lyCx2AbK9LMshJu.png)

## OCR流程

------

1. 采集
2. 预处理，校正
3. 分割成独立连通域
   ![img](https://s2.loli.net/2025/06/12/JWKbH92TwgDNpmA.png)
   如果每个字没有形成独立连通域（如上图），需要使用形态学，使每个汉字的各笔画在各自的一个连通域内，不同的汉字的各笔画在不同的连通域内。如“明≠日月”的关系
4. 使用现成的，或训练
   - trf文件：文本与字符的关联文件，只是对应关系。可被OCR助手和OCR读取函数读取
   - omc文件：为训练后的文件，可用于识别。不能被OCR助手读取，只能被orc读取函数读取
5. 识别

## 色彩识别

------

1. RGB转HSV

   色调（H），饱和度（S），明度（V）

   主要分析H，S分量
   ![img](https://s2.loli.net/2025/06/12/NdvZfQGreTAU9iF.png)

2. 分类器（另外详细介绍）

## 对比度相关算子

------

- `scale_image`
- `emphasize`
- `gray_range_rect`等（灰度图的形态学变换）
- `equ_histo_image`直方图均衡化

## 几大“门派”总结

------

1. 基础理论
2. 灰度变换
3. 增强
4. 几何变换
5. 分割
6. 频域处理
7. 形态学
8. 复原
9. 运动图像
10. 图像配准
