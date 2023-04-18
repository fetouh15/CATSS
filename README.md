# CATSS (Cerebral Arterial Tree Segmentation Software)

<h2> Welcome to CATSS! The only open-source cerebral brain vessel multi-class segmentation tool on the market! <img 
 </h2>

 <h3> What is CATSS?</h3>
 
 Time of Flight Magnetic resonance angiography (TOF-MRA) is a commonly used imaging technique in clinical and research environments to examine the cerebral arterial tree. However, analyzing the whole brain arterial vasculature requires skilled experts to intervene ~~manually~~, making it time-consuming and inconsistent, especially for multiclass segmentations. To address this, we developed CATSS a fully automated method that segments the cerebral arterial tree, and labels its arteries.
 
![image](https://user-images.githubusercontent.com/38469694/232742988-f021b39c-3867-4731-bdb4-65b63d1429b9.png)

 <h3> How does it work? </h3>
 All CATSS needs is the **TOF-MRA** [.nii.gz], and a corresponding **CW labels** [.nii.gz]. CATSS generates an binary segmentation of the whole arterial tree, and then uses CW labels as seed points to label the arterial tree accordingly.
