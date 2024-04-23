# Camera Calibration
> It integrates multiple script tools related to camera calibration to facilitate the completion of a complete vehicle surround view camera calibration process.  
> Each code file can be used independently, and external interfaces are also provided for calling 
  
![](https://img.shields.io/badge/Language-python-blue.svg) 　
![](https://img.shields.io/badge/Requirement-openCV-brightgreen) 　
![License](https://img.shields.io/badge/License-GPL-orange.svg)

## DEMO
![DEMO](demo.gif)

## Quick Start
Clone the repository and run main.py to view the simple example results  
Make sure you have installed **opencv(>=3.4.2)** and **numpy(>=1.19.2)**
```
git clone https://github.com/dyfcalid/CameraCalibration.git
cd ./CameraCalibration
python main.py
```  
  
## File Tree  
> Project structure preview  
```
│  main.py                    // main program
│
├─ExtrinsicCalibration
│  │  extrinsicCalib.ipynb    // External parameter calibration code (including comments)
│  │  extrinsicCalib.py       // External parameter calibration python code
│  │  README.md               // External parameter calibration document
│  │  __init__.py             // init file, API description
│  │
│  └─data                     // External parameter calibration data folder
│
├─IntrinsicCalibration
│  │  intrinsicCalib.ipynb    // Internal parameter calibration code (including comments)
│  │  intrinsicCalib.py       // Internal parameter calibration python code
│  │  README.md               // Internal reference calibration document
│  │  __init__.py             // init file, API description
│  │
│  └─data                     // Internal reference calibration data folder
│
├─SurroundBirdEyeView
│  │  surroundBEV.ipynb       // Look around the code (with comments)
│  │  surroundBEV.py          // A bird's eye view of python code
│  │  README.md               // Look around the document
│  │  __init__.py             // init file, API description
│  │
│  └─data                     // Look around the bird's eye view parameter folder
│     ├─front                 // Store front camera K, D, H parameter files
│     ├─back                  // Store the rear camera K, D, H parameter files
│     ├─left                  // Store left camera K, D, H parameter files
│     └─right                 // Store the K, D, H parameter files of the right camera
│
└─Tools                       // Some related calibration tools
    │  collect.py             // Image Acquisition
    │  undistort.py           // Image dedistortion
    └─data                    // data folder

```

  
## Camera Intrinsic Calibration 
> Camera internal parameter calibration   
  
`intrinsicCalib.py`  [View documentation](./IntrinsicCalibration/README.md/)  
Including **online calibration** and **offline calibration** of the camera, including **fisheye camera** and **ordinary camera** models，  
并支持**相机、视频、图像**三种输入，生成相机内参和畸变向量   

- 可以直接运行python文件，并通过argparse输入更多参数，argparse参数表详见文档
```
python intrinsicCalib.py
```  

- 此外，提供`InCalibrator`类供调用，使用说明如下，具体示例见**main.py**  
```
from intrinsicCalib import InCalibrator

calibrator = InCalibrator(camera_type)              # 初始化内参标定器
for img in images:
    result = calibrator(img)                        # 每次读入一张原始图片 更新标定结果
undist_img = calibrator.undistort(raw_frame)        # 使用undistort方法得到去畸变图像
```
或者调用`CalibMode`类，使用预设好的标定模式，各模式详见文档  
```
from intrinsicCalib import InCalibrator, CalibMode

calibrator = InCalibrator(camera_type)              # 初始化内参标定器
calib = CalibMode(calibrator, input_type, mode)     # 选择标定模式
result = calib()                                    # 开始标定
```
可以直接修改原文件中的各参数，或使用`get_args()`方法获取参数并修改
```
args = InCalibrator.get_args()                      # 获取args参数
args.INPUT_PATH = './IntrinsicCalibration/data/'    # 修改args参数
calibrator = InCalibrator(camera_type)              # 初始化内参标定器
```  

示例结果：  
<img src="https://i.loli.net/2021/06/22/nxOsU1mM4D3kJWS.png" width="750" height="200" alt="inCalib_result.jpg"/>  
<img src="https://i.loli.net/2021/06/22/iVETOUIMqCRHDYr.png" width="750" height="300" alt="inCalib_image.jpg"/>  
  
  
## Camera Extrinsic Calibration  
> 相机外参标定   

`extrinsicCalib.py`  [查看文档](./ExtrinsicCalibration/README.md/)    
完成相机的**外参标定**，实现**任意两个视图（包含相同标定板）的转换**，生成**单应性变换矩阵**  
如：基于**无人机相机**和**车载环视相机**同时拍摄地面的标定板，进行车载相机的外参标定，  
生成车载相机至无人机相机的单应性变换矩阵，实现**鸟瞰图**的转换（即将车载相机视角转换至无人机视角）    
  
- 可以直接运行python文件，并通过argparse输入更多参数，argparse参数表详见文档
```
python extrinsicCalib.py
```  
  
- 此外，提供`ExCalibrator`类供调用，使用说明如下，具体示例见**main.py**  
```
from extrinsicCalib import ExCalibrator

exCalib = ExCalibrator()                            # 初始化外参标定器
homography = exCalib(src_raw, dst_raw)              # 输入对应的两张去畸变图像 得到单应性矩阵
src_warp = exCalib.warp()                           # 使用warp方法得到原始图像的变换结果
```    
可以直接修改原文件中的各参数，或使用`get_args()`方法获取参数并修改  
```
args = ExCalibrator.get_args()                      # 获取args参数
args.INPUT_PATH = './ExtrinsicCalibration/data/'    # 修改args参数
exCalib = ExCalibrator()                            # 初始化外参标定器
```    
  
示例结果：   
![exCalib_result.jpg](https://i.loli.net/2021/06/22/5fMmcxTuZ2aIUyN.png)   
  
  
## Surround Camera Bird Eye View  
> 环视相机鸟瞰拼接图生成  
  
`surroundBEV.py`  [查看文档](./SurroundBirdEyeView/README.md/)    
输入前后左右四张**原始相机图像**，生成**鸟瞰图**  
包括**直接拼接**和**融合拼接**，并可以进行**亮度平衡和白平衡**   
  
- 可以直接运行python文件，并通过argparse输入更多参数，argparse参数表详见文档
```
python surroundBEV.py
```  
  
- 此外，提供`BevGenerator`类供调用，使用说明如下，具体示例见**main.py**  
```
from surroundBEV import BevGenerator

bev = BevGenerator()                                # 初始化环视鸟瞰生成器
surround = bev(front,back,left,right)               # 输入前后左右四张原始相机图像 得到拼接后的鸟瞰图
```    
上面生成的是直接拼接的结果，能够保证**实时性**，此外也可以使用融合和平衡，但速度较慢，如
```
bev = BevGenerator(blend=True, balance=True)        # 使用图像融合以及平衡
surround = bev(front,back,left,right,car)           # 可以加入车辆图片
```
可以直接修改原文件中的各参数，或使用`get_args()`方法获取参数并修改  
```
args = BevGenerator.get_args()                      # 获取环视鸟瞰args参数
args.CAR_WIDTH = 200
args.CAR_HEIGHT = 350                               # 修改为新的参数
bev = BevGenerator()                                # 初始化环视鸟瞰生成器
```    
  
示例结果：    
<div align=center><img src="https://i.loli.net/2021/06/22/fOwPsTYkCFeo8dW.png" width="740" height="170" alt="camera.jpg"/></div>  
<div align=center><img src="https://i.loli.net/2021/06/22/HeKJVBm2vEINy4z.png" width="360" height="400" alt="bev.jpg"/></div>   
   
   
## Other Tools  
用`collect.py`可以开启相机完成图像或视频的**数据采集**  
用`undistort.py`可以批量完成图像的**去畸变处理**   
用`decomposeH.py`可以由单应性矩阵H和相机内参K得到**旋转矩阵R和平移矩阵T** （有多个结果需要筛选）   
用`timeAlign.py`可以将以**时间戳**命名的图片按时间**对准**，得到对应的列表   
用`img2vid.py`可以将图片转化为视频  
     
## License  
[GPL-3.0 License](LICENSE)  
  
  
*`Copyright (c) 2021 ZZH`*  
  
  
