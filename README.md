# CATSS (Cerebral Arterial Tree Segmentation Software)

<h2> Welcome to CATSS! The only open-source cerebral brain vessel multi-class segmentation tool on the market! <img 
 </h2>

 <h3> What is CATSS?</h3>
 
 Time of Flight Magnetic resonance angiography (TOF-MRA) is a commonly used imaging technique in clinical and research environments to examine the cerebral arterial tree. However, analyzing the whole brain arterial vasculature requires skilled experts to intervene ~~manually~~, making it time-consuming and inconsistent, especially for multiclass segmentations. To address this, we developed CATSS a fully automated method that segments the cerebral arterial tree, and labels its arteries.
 
![image](https://user-images.githubusercontent.com/38469694/232742988-f021b39c-3867-4731-bdb4-65b63d1429b9.png)

 <h3> How does it work? </h3>
 All CATSS needs is the __TOF-MRA__ [.nii.gz], and a corresponding __CW labels__ [.nii.gz]. CATSS generates an binary segmentation of the whole arterial tree, and then uses CW labels as seed points to label the arterial tree accordingly.

 <h3> What does it work on? </h3>
 
 <h3> How to use it? </h3>
CATSS uses the following labeling scheme:
 
 Make sure your CW labels use the same labeling scheme to get flawless results!
 CATSS uses an I/O directory to store all the inputs and outputs.
 
 
 
 CATSS is on dockerhub, its plug&play!
 
