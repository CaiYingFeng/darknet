# 新增接口
+ 接口描述
头文件：include/darknet.h
库文件：libdarknet.a、libdarknet.so
```c

network *get_load_network(char *cfgfile,char *weightfile);  //预加载网络
det_box *yolo_detect(image im,network* net,float thresh);   //开始检测，必须在加载网络之后
```
+ 输入值
```
char *cfgfile,  //设置文件路径
char *weightfile,   //网络模型文件路径

image im,
network* net    //指向网络模型的指针
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
+ darknet.h增加yolo_detect函数描述，写入src/demo.c中
+ darknet.h增加get_load_network函数描述，写入src/demo.c中
+ demo.c中默认config如下，执行会遇到路径问题，修改之后重修编译。
- [ ] todo：将这些参数放入config文件。
```c
    //configure
    char *datacfg="cfg/coco.data";
    float hier_thresh=0.5;
    char *name_list = option_find_str(options, "names", "data/names.list");
```
### usage
0. 编译
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
    network *net=get_load_network("cfg/yolov3.cfg","/home/shy/workspace/darknet_wu/yolov3.weights");
    det_box *dets=yolo_detect(im,net,0.5);
    
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
include_directories( "~/workspace/darknet/include")
include_directories( "~/workspace/darknet/src") #for image.h
link_directories("~/workspace/darknet/")

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
[参考库](https://github.com/wuxiaolang/darknet)
---
## YOLO
+ 官网：https://pjreddie.com/darknet/yolo/
+ github：https://github.com/pjreddie/darknet



