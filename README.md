# 新增接口
+ 接口描述
头文件：include/darknet.h
库文件：libdarknet.a、libdarknet.a
```c
det_box *yolo_detect(char *cfgfile,char *weightfile,image im,float thresh)
```
+ 输入值
```
char *cfgfile,
char *weightfile,
image im,
float thresh

```
+ 返回数据格式
```c
//返回det_box数组，单位数据结构如下：
typedef struct{
    float classs_id, x_min, y_min, w,h,p；
} det_box;
```
### change log
darknet.h增加yolo_detect函数描述，写入src/demo.c中
### usage
1. CMakeList链接静态库
2. include头文件
```c
#include "image_opencv.h"   //mat2image
extern "C" {
#include "darknet.h"
}
```
### 示例
```cpp
//main.cpp
#include <opencv2/core/core.hpp>
#include <opencv2/features2d/features2d.hpp>
#include <opencv2/highgui/highgui.hpp>
#include<stdio.h>

#include "image_opencv.h"
extern "C" {
#include "darknet.h"
}
using namespace cv;

int main(){
    Mat mat=imread("dog.jpg");
    image im=mat_to_image(mat);
    det_box *dets=yolo_detect("cfg/yolov3.cfg","/home/shy/workspace/darknet_wu/yolov3.weights",im,0.5);
    
    printf("%f %f %f %f %f %f ",dets[0].class_id,dets[0].x_min,dets[0].y_min,dets[0].w,dets[0].h,dets[0].p);
    return 0;
}

```


```c make
#CMakeList.txt
cmake_minimum_required( VERSION 2.8 )
project(testAPI)

set( CMAKE_BUILD_TYPE Release )
set( CMAKE_CXX_FLAGS "-std=c++11 -O3" )

# opencv 
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )

#set path
include_directories( "~/workspace/darknet_wu/include")
include_directories( "~/workspace/darknet_wu/src") #for image.h
link_directories("~/workspace/darknet_wu/")

#add_library 设定image_opencv.cpp的绝对路径
add_library(image_opencv_s SHARED ~/workspace/darknet_wu/src/image_opencv.cpp)

add_executable(test1 main.cpp)
target_link_libraries( test1 ${OpenCV_LIBS} image_opencv_s darknet )
```
```sh
shy@shy-HP:./test1
Predicted in 0.067707 seconds.
dog: 100%
truck: 78%
bicycle: 98%
16.000000 121.000000 228.000000 197.031525 283.358337 0.000000
```
---
[原仓库](https://github.com/wuxiaolang/darknet)
# 批量模式
## 0. 主要工作：
+ 在 [**darknet.c**](https://github.com/wuxiaolang/darknet/blob/master/examples/darknet.c?1552372879559) 中添加了 `detect_tum_batch` 命令；
+ 在 [**detector.c**](https://github.com/wuxiaolang/darknet/blob/master/examples/detector.c?1552372948327) 中添加了**读取数据集图片**的函数 `test_detector_tum_batch()`；
+ 在 [**image.c**](https://github.com/wuxiaolang/darknet/blob/master/src/image.c?1552373025166) 中添加了绘制与**保存结果**的函数 `draw_save_detections()`。
+ 代码：[**https://github.com/wuxiaolang/darknet**](https://github.com/wuxiaolang/darknet)

## 1. 指令与效果
+ **TUM 数据集**    
指令：
```
./darknet detect_tum_batch [path_of_cfg] [path_of_weight] [path_of_dataset] [output_folder] [-thresh thresh]
```
&emsp;&emsp;例如：
```
./darknet detect_tum_batch cfg/yolov3.cfg yolov3.weights /home/wu/data/dataset/rgbd_dataset_freiburg1_desk/ /home/wu/data/ -thresh 0.4
```
&emsp;&emsp;注意：请在 [output_folder] 下创建 `yolo_imgs` 和 `yolo_txts` 文件夹来存放绘制了检测框的图片和检测框 txt 信息。

+ **其他数据集**：可仿照改动的函数写，后续用到的时候再添加其他数据集命令。

## 2. 结果
+ **以下所有的输出格式、内容可以在 `detector.c` 的 `test_detector_tum_batch()` 函数中修改。**
+ 输出的图片保存在 `output_folder/yolo_imgs` 路径下，文件名为：`[时间戳].jpg`，例如 `1905140001.223000.jpg`；
+ 输出的 txt 文件保存在 `output_folder/yolo_txts` 路径下，文件名为：`[时间戳].txt`，例如 `1905140001.223000.txt`；
+ txt 保存格式：类别 ID &emsp; 左上角.x &emsp; 左上角.y &emsp; 长 &emsp; 宽 &emsp; 置信度
    ```
    66   70    294   260   185    0.794691
    64   255   258   63     38    0.813716
    62   224   0     243   234    0.610995
    ```
---
## YOLO
+ 官网：https://pjreddie.com/darknet/yolo/
+ github：https://github.com/pjreddie/darknet



