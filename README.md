# CATSS (Cerebral Arterial Tree Segmentation Software) 
 

## Welcome to CATSS! The only open-source cerebral brain vessel multi-class segmentation tool on the market! 
<img align="right" width="200" height="200" src="https://user-images.githubusercontent.com/38469694/232968254-69a4f022-9def-43ac-86e9-ee1c4b84a9b9.jpeg"> 

<h2> What is CATSS?</h2>



 Time of Flight Magnetic resonance angiography (TOF-MRA) is a commonly used imaging technique in clinical and research environments to examine the cerebral arterial tree. However, analyzing the whole brain arterial vasculature requires skilled experts to intervene ~~manually~~, making it time-consuming and inconsistent, especially for multiclass segmentations. To address this, we developed CATSS a fully automated method that segments the cerebral arterial tree, and labels its arteries.
 

 <h2> How does it work? </h2>
 All CATSS needs is the __TOF-MRA__ [.nii.gz], and a corresponding __CW labels__ [.nii.gz]. CATSS generates an binary segmentation of the whole arterial tree, and then uses CW labels as seed points to label the arterial tree accordingly.

 <h2> What does it work on? </h2>
 
 ![image](https://user-images.githubusercontent.com/38469694/232742988-f021b39c-3867-4731-bdb4-65b63d1429b9.png)

 <h2> How to use it? </h2>
CATSS uses the following labeling scheme:
 
 Make sure your CW labels use the same labeling scheme to get flawless results!
 
 CATSS uses an I/O directory to store all the inputs and outputs. The I/O directory must contain a the CW labels file **TOF_CW.nii.gz** and a subdirectory **nnio** with an input TOF file. The input file must end with _0000 and have an extension of .nii.gz such as in the example below, **TOF_resampled_0000.nii.gz**.
 
 
 
 
![image](https://user-images.githubusercontent.com/38469694/232751654-642bc4f9-e8c7-45f0-befc-6217dc030daa.png)
 
 CATSS is on dockerhub, its plug&play!
 Simply use **sudo docker run --gpus all -v '/absolute/path/to/io_directory':/io_directory fetouh15/catss --rm -it**
to install and run CATSS
