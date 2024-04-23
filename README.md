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
It also supports three inputs: camera, video, and image to generate internal camera parameters and distortion vectors.
- You can directly run the python file and enter more parameters through argparse. See the documentation for the argparse parameter list.
```
python intrinsicCalib.py
```  

- In addition, the `InCalibrator` class is provided for calling. The usage instructions are as follows. For specific examples, see **main.py**
```
from intrinsicCalib import InCalibrator
始
calibrator = InCalibrator(camera_type)              # Initialize the internal parameter calibrator
for img in images:
    result = calibrator(img)                        # Each time an original image is read in, the calibration results are updated.
undist_img = calibrator.undistort(raw_frame)        # Use the undistort method to get the dedistorted image
```
Or call the `CalibMode` class to use the preset calibration mode. Please see the documentation for each mode.
```
from intrinsicCalib import InCalibrator, CalibMode

calibrator = InCalibrator(camera_type)              # Initialize internal parameter calibrator
calib = CalibMode(calibrator, input_type, mode)     # Select calibration mode
result = calib()                                    # Start calibration
```
You can directly modify each parameter in the original file, or use the `get_args()` method to obtain the parameters and modify them.
```
args = InCalibrator.get_args()                      # Get args parameters
args.INPUT_PATH = './IntrinsicCalibration/data/'    # Modify args parameters
calibrator = InCalibrator(camera_type)              # Initialize internal parameter calibrator
```  

Example results：  
<img src="https://i.loli.net/2021/06/22/nxOsU1mM4D3kJWS.png" width="750" height="200" alt="inCalib_result.jpg"/>  
<img src="https://i.loli.net/2021/06/22/iVETOUIMqCRHDYr.png" width="750" height="300" alt="inCalib_image.jpg"/>  
  
  
## Camera Extrinsic Calibration  
> Camera external parameter calibration   

`extrinsicCalib.py` [View documentation](./ExtrinsicCalibration/README.md/)
Complete the **external parameter calibration** of the camera, realize the conversion** of any two views (including the same calibration plate), and generate the **homography transformation matrix**
For example: based on the **drone camera** and **vehicle surround camera** shooting the calibration plate on the ground at the same time, the external parameters of the vehicle camera are calibrated,
Generate the homography transformation matrix from the vehicle camera to the drone camera to achieve the conversion of **bird's eye view** (that is, convert the vehicle camera perspective to the drone perspective)

  
- You can directly run the python file and enter more parameters through argparse. See the documentation for the argparse parameter list.
```
python extrinsicCalib.py
```  
  
- In addition, the `ExCalibrator` class is provided for calling. The usage instructions are as follows. For specific examples, see **main.py**

```
from extrinsicCalib import ExCalibrator

exCalib = ExCalibrator()                            # Initialize the external parameter calibrator
homography = exCalib(src_raw, dst_raw)              # Input the corresponding two dedistorted images to obtain the homography matrix
src_warp = exCalib.warp()                           # Use the warp method to get the transformation result of the original image
```    
You can directly modify each parameter in the original file, or use the `get_args()` method to obtain the parameters and modify them.
```
args = ExCalibrator.get_args() # Get args parameters
args.INPUT_PATH = './ExtrinsicCalibration/data/' # Modify args parameters
exCalib = ExCalibrator() # Initialize the external parameter calibrator
```
  
Example results：   
![exCalib_result.jpg](https://i.loli.net/2021/06/22/5fMmcxTuZ2aIUyN.png)   
  
  
## Surround Camera Bird Eye View
> Generate bird's-eye view stitching from surround-view camera
  
`surroundBEV.py` [View documentation](./SurroundBirdEyeView/README.md/)
Input four **original camera images** from the front, back, left and right to generate a **bird's eye view**
Including **direct splicing** and **fusion splicing**, and can perform **brightness balance and white balance**
  
- You can directly run the python file and enter more parameters through argparse. See the documentation for the argparse parameter list.
```
python surroundBEV.py
```
  
- In addition, the `BevGenerator` class is provided for calling. The usage instructions are as follows. For specific examples, see **main.py**
```
from surroundBEV import BevGenerator

bev = BevGenerator() # Initialize the bird's-eye view generator
surround = bev(front,back,left,right) # Input the four original camera images of the front, back, left and right to get the spliced bird's-eye view.
```
What is generated above is the result of direct splicing, which can ensure **real-time**. In addition, fusion and balancing can also be used, but the speed is slower, such as
```
bev = BevGenerator(blend=True, balance=True) # Use image fusion and balance
surround = bev(front,back,left,right,car) # You can add vehicle pictures
```
You can directly modify each parameter in the original file, or use the `get_args()` method to obtain the parameters and modify them.
```
args = BevGenerator.get_args() # Get the bird's-eye view args parameters
args.CAR_WIDTH = 200
args.CAR_HEIGHT = 350 # Modify to new parameters
bev = BevGenerator() # Initialize the bird's-eye view generator
```
  
Example results:
<div align=center><img src="https://i.loli.net/2021/06/22/fOwPsTYkCFeo8dW.png" width="740" height="170" alt="camera.jpg"/> </div>
<div align=center><img src="https://i.loli.net/2021/06/22/HeKJVBm2vEINy4z.png" width="360" height="400" alt="bev.jpg"/> </div>
   
   
## Other Tools
Use `collect.py` to open the camera to complete **data collection** of images or videos.
Use `undistort.py` to complete the **de-distortion processing** of images in batches
Use `decomposeH.py` to get the **rotation matrix R and translation matrix T** from the homography matrix H and camera internal parameters K (there are multiple results that need to be filtered)
Use `timeAlign.py` to **align** pictures named with **timestamp** by time and get the corresponding list
Use `img2vid.py` to convert images into videos
     
## License
[GPL-3.0 License](LICENSE)
  
  
*`Copyright (c) 2021 ZZH`*
