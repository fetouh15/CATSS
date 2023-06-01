# CATSS (Cerebral Arterial Tree Segmentation Software) 
 

## Welcome to CATSS! The only open-source cerebral brain vessel multi-class segmentation tool on the market! 
<img align="right" width="200" height="200" src="https://user-images.githubusercontent.com/38469694/232968254-69a4f022-9def-43ac-86e9-ee1c4b84a9b9.jpeg"> 

## What is CATSS?
 Time of Flight Magnetic resonance angiography (TOF-MRA) is a commonly used imaging technique in clinical and research environments to examine the cerebral arterial tree. However, analyzing the whole brain arterial vasculature requires skilled experts to intervene ~~manually~~, making it time-consuming and inconsistent, especially for multiclass segmentations. To address this, we developed CATSS a fully automated method that segments the cerebral arterial tree, and labels its arteries.
 

## How does it work? 
 All what CATSS needs is a TOF-MRA nifti file, and a corresponding CW labeled segmentation. CATSS generates an binary segmentation of the whole arterial tree, and then uses CW labels as seed points to label the arterial tree accordingly.
 
![image](https://user-images.githubusercontent.com/38469694/232989548-eeceee59-68d2-4c86-903b-6c910bd9cacb.png)

## Prerequisites
A CATSS docker image is available on Dockerhub, so its just plug&play! CATSS only needs [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) if you plan on runnig it on a NVIDIA GPU.
Please follow [the installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).
> :warning: For GPUs only cuda is supported.
 
 ## What does it work on? 
Well, **EVERYTHING!** Whether the TOF-MRA is 1.5T, 3T, 7T, small FOV, or large FOV, it will work! CATSS can segment skull stripped images, and images with venetian blind artifacts.
 
![image](https://user-images.githubusercontent.com/38469694/232984119-e5e6729c-e54e-46e9-91c8-1720bd1c0dda.png)

## How to use it? 
**CATSS uses the same labeling scheme for [eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab):**

 | Arteries | Left | Right |
|----------|------|:-----:|
| ICA      | 1    |   2   |
| BAS      | 3   |  3  |
| AComm      | 4   |  4  |
| ACA-A1   | 5    |   6   |
| MCA-M1   | 7    |   8   |
| PComm    | 9    |  10   |
| PCA-P1   | 11   |  12   |
| PCA-P2   | 13   |  14   |
| SCA      | 15   |  16   |
| AChA     | 17   |  18   |

> :warning: **Users are not required to use eICAB to produce the CW labels; however it is recommended. In case of using any other method, users must make sure that their CW labels match the previous labeling scheme as CATSS will only work with the above labeling grid.**


 CATSS uses an **I/O** directory to store all the inputs and outputs. The I/O directory must contain only one CW labels file ending with **_CW.nii.gz** and only one TOF-MRA file ending with **_TOF.nii.gz**. 
 
 CATSS generates 3 output files: 
 + **TOF_binary_segmentation_raw.nii.gz**: Binary segmentation of the cerebral arterial tree without postprocessing
 + **TOF_binary_segmentation.nii.gz**: Binary segmentation of the cerebral arterial tree
 + **TOF_multiclass_segmentation.nii.gz**: Multi-Class segmentation of the cerebral arterial tree
### Here  is a snapshot of the IO directory with the required input files before running CATSS:
![image](https://user-images.githubusercontent.com/38469694/233215344-fda9b878-1c25-4b75-acb4-3bdfc45b72e7.png)

### Here is a snapshot of the IO directory after running CATSS:
![image](https://user-images.githubusercontent.com/38469694/233215363-c6ed6047-31e8-4cf0-b217-c85490739266.png)
 
# Will be released by end of 2023 !!!!!
