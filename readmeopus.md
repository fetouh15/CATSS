<div align="center">

<img width="40%" alt="60994299-a1d0-44f6-821d-1a7e0b72f09e copy" src="https://github.com/user-attachments/assets/68d732e3-d53b-4336-a4eb-e375281d2b57" />


**Cerebral Arterial Tree Segmentation Software**

Fully automated, multi-class segmentation of the whole cerebral arterial tree in TOF-MRA.

<!-- TODO(karim): when the paper is out, point the Paper badge at its DOI and change "coming soon" to the journal name. -->
<!-- TODO(karim): once you pick a license, change "TBD" in the License badge, e.g. https://img.shields.io/badge/license-MIT-green -->

[![Paper](https://img.shields.io/badge/paper-coming%20soon-lightgrey)](#citation)
[![ISMRM 2023](https://img.shields.io/badge/ISMRM%202023-abstract%205271-1f6feb)](https://archive.ismrm.org/2023/5271.html)
[![Docker](https://img.shields.io/badge/run%20with-Docker-2496ED?logo=docker&logoColor=white)](#quick-start)
[![License](https://img.shields.io/badge/license-TBD-lightgrey)](#license)

[Quick start](#quick-start) •
[Inputs](#inputs) •
[Outputs](#outputs) •
[Label scheme](#label-scheme) •
[Validation](#validation) •
[Troubleshooting](#troubleshooting) •
[Citation](#citation)

</div>

---

<p align="center">
<img width="2752" height="1536" alt="Gemini_Generated_Image_vvyl8vvvyl8vvvyl" src="https://github.com/user-attachments/assets/37cf7ca2-da9c-4292-a721-65a94af17ede" /></p>

## Overview

Time-of-flight magnetic resonance angiography (TOF-MRA) is widely used in clinical and research settings to examine the cerebral arteries. Analysing the whole arterial tree, though, still means experts tracing and labelling vessels by hand. That is slow, it varies from one reader to the next, and it does not scale to large studies, especially when every artery needs its own label.

CATSS does it automatically. Give it a TOF-MRA and a Circle of Willis (CW) labelling, and it returns:

1. a **binary segmentation** of the whole cerebral arterial tree, produced by a deep-learning model ([nnU-Net](https://github.com/MIC-DKFZ/nnUNet)), and
2. a **multi-class segmentation**, where each vessel gets the label of the CW artery it branches from, using the CW labels as seed points.

In [our paper](#citation), CATSS's artery labels were used to map each participant's cerebral border zones (the regions between arterial territories) and to study where white matter hyperintensities accumulate with age.

**Highlights**

- **Fully automated.** No manual seeding, thresholding or correction.
- **Multi-class.** Uses the same 18-label scheme as [eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab), extended from the Circle of Willis out to the distal arteries.
- **Accurate.** Agrees with an expert's manual annotation about as well as the expert agrees with themself (see [Validation](#validation)).
- **Fast.** About 10 minutes per scan.
- **Works on varied data.** 1.5T, 3T and 7T; small and large fields of view; skull-stripped or not; images with venetian-blind artifacts.
- **Easy to install.** Ships as a Docker image, so there is nothing to install by hand except Docker.

<p align="center">
  <img src="https://user-images.githubusercontent.com/38469694/232984119-e5e6729c-e54e-46e9-91c8-1720bd1c0dda.png" width="90%" alt="CATSS results on TOF-MRAs at different field strengths, fields of view and with artifacts">
</p>

## News

<!-- TODO(karim): add the paper line when it is published. -->
- **2023**: CATSS presented at ISMRM 2023 ([abstract #5271](https://archive.ismrm.org/2023/5271.html)).

## Requirements

- **Docker.** [Docker Engine](https://docs.docker.com/engine/install/) on Linux, or Docker Desktop on Windows (WSL 2) or macOS.
- **An NVIDIA GPU (optional, recommended)** with the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). Only CUDA (NVIDIA) GPUs are supported.

Without a GPU, CATSS runs on the CPU, just more slowly. On macOS, Docker has no GPU access, so CATSS always runs on the CPU there.

## Quick start

**1. Pull the image**

<!-- TODO(karim): replace <docker-image> with the real Docker Hub name and tag, e.g. fetouh15/catss:1.0 -->
```bash
docker pull <docker-image>
```

**2. Put your two input files in one folder**

```text
sub-01/
├── sub-01_TOF.nii.gz    # TOF-MRA
└── sub-01_CW.nii.gz     # Circle of Willis labels (see "Inputs")
```

**3. Run CATSS on that folder**

<!-- TODO(karim): replace /io with the path CATSS reads from inside the container -->
```bash
docker run --rm --gpus all \
  -v /absolute/path/to/sub-01:/io \
  <docker-image>
```

The results are written to the same folder. To run on the CPU, remove `--gpus all`.

<details>
<summary><b>Processing a whole study</b></summary>

Give each subject its own folder, then loop over them:

```bash
for subj in /path/to/study/sub-*/; do
  docker run --rm --gpus all -v "${subj%/}":/io <docker-image>
done
```

</details>

## Inputs

CATSS reads from and writes to one **I/O folder** per subject. The folder must contain **exactly one** of each:

| File name ends with | Content |
|---|---|
| `_TOF.nii.gz` | The TOF-MRA, as compressed NIfTI |
| `_CW.nii.gz` | Circle of Willis labels for the same scan, on the same voxel grid as the TOF (same dimensions and orientation) |

### Getting the Circle of Willis labels

We recommend producing the CW labels with **[eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab)**, which uses exactly the label scheme CATSS expects.

> [!IMPORTANT]
> You can use another method, but its labels **must** match the [label scheme](#label-scheme) below. CATSS only understands those label values.

## Outputs

CATSS writes three files into the I/O folder:

| File | Content |
|---|---|
| `TOF_binary_segmentation_raw.nii.gz` | Binary segmentation of the arterial tree, straight from the network, without post-processing |
| `TOF_binary_segmentation.nii.gz` | Binary segmentation of the arterial tree, after post-processing |
| `TOF_multiclass_segmentation.nii.gz` | Multi-class segmentation of the arterial tree, using the [label scheme](#label-scheme) below |

After a run, the folder looks like this:

```text
sub-01/
├── sub-01_TOF.nii.gz
├── sub-01_CW.nii.gz
├── TOF_binary_segmentation_raw.nii.gz
├── TOF_binary_segmentation.nii.gz
└── TOF_multiclass_segmentation.nii.gz
```

### Viewing the results

Any NIfTI viewer works. For example, with [ITK-SNAP](http://www.itksnap.org):

```bash
itksnap -g sub-01/sub-01_TOF.nii.gz -s sub-01/TOF_multiclass_segmentation.nii.gz
```

[3D Slicer](https://www.slicer.org) and [FSLeyes](https://open.win.ox.ac.uk/pages/fsl/fsleyes/fsleyes/userdoc/) also work well, especially for 3D renderings of the tree.

## Label scheme

CATSS uses the same labels as eICAB. The same values appear in the CW input and in the multi-class output. `0` is background.

| Artery | Full name | Left | Right |
|---|---|:---:|:---:|
| ICA | Internal carotid artery | 1 | 2 |
| BAS | Basilar artery | 3 | 3 |
| AComm | Anterior communicating artery | 4 | 4 |
| ACA-A1 | Anterior cerebral artery, A1 segment | 5 | 6 |
| MCA-M1 | Middle cerebral artery, M1 segment | 7 | 8 |
| PComm | Posterior communicating artery | 9 | 10 |
| PCA-P1 | Posterior cerebral artery, P1 segment | 11 | 12 |
| PCA-P2 | Posterior cerebral artery, P2 segment | 13 | 14 |
| SCA | Superior cerebellar artery | 15 | 16 |
| AChA | Anterior choroidal artery | 17 | 18 |

BAS and AComm are midline arteries, so they have one label each.

## Validation

<p align="center">
  <img src="assets/catss_validation.png" width="60%" alt="A: TOF-MRA maximum intensity projection and its CATSS segmentation, with ACA in green, MCA in yellow and PCA in red. B: Dice scores against an expert's manual annotations for the MCA, ACA and PCA. C: Dice scores between two scans of the same person versus scans of different people.">
</p>

CATSS was compared with an expert's manual annotations and tested for test-retest reliability. The figure and numbers are from the paper.

| Test | Dice score |
|---|---|
| Whole arterial tree vs. expert annotation (10 scans) | **73.1% ± 2.66%** |
| ACA territory vs. expert annotation | 67.9% ± 5.2% |
| MCA territory vs. expert annotation | 68.6% ± 3.5% |
| PCA territory vs. expert annotation | 58.6% ± 4.7% |
| The same expert annotating the same scan twice, two years apart | 71.5% |
| The same person scanned twice (10 subjects, Midnight Scan Club) | 71.5% ± 2.71% |

CATSS agrees with the expert about as closely as the expert agrees with themself, so it is close to the practical ceiling of manual tracing. Two scans of the same person also give far more similar segmentations than scans of two different people (panel C).

The model was trained and tested on 126 TOF-MRAs from four sources: 31 from the CR-CHUS (Sherbrooke), 20 from MIDAS, 10 from the Midnight Scan Club and 65 from OASIS-3. Of these, 89 were used for training, 27 for testing and 10 for validation.

## Troubleshooting

<details>
<summary><code>could not select device driver "" with capabilities: [[gpu]]</code></summary>

Docker cannot see your GPU. Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html), restart Docker, and check that `nvidia-smi` works on the host. Or remove `--gpus all` to run on the CPU.
</details>

<details>
<summary>CATSS does not find my input files</summary>

- The folder must contain exactly **one** file ending in `_TOF.nii.gz` and **one** ending in `_CW.nii.gz`. The suffixes are case-sensitive, and `.nii` (uncompressed) files are not picked up.
- Use an **absolute** path in `-v`, for example `-v "$(pwd)/sub-01":/io`, not `-v sub-01:/io`.
- On Windows, run the command from WSL 2 and use the WSL path to the folder.
</details>

<details>
<summary>The multi-class labels look wrong</summary>

- Check that your CW labels use the [label scheme](#label-scheme) above. Labels from another tool often use different values.
- Check that the CW labels sit on the same voxel grid as the TOF. Open both in ITK-SNAP: they should overlap exactly.
</details>

<details>
<summary>The output files are owned by root</summary>

That is normal for Docker on Linux. Take them back with `sudo chown -R "$USER" /path/to/sub-01`.
</details>

Still stuck? [Open an issue](https://github.com/fetouh15/CATSS/issues/new/choose) and include the command you ran and the full error output.

## Citation

If you use CATSS in your research, please cite:

<!-- TODO(karim): replace with the journal paper's BibTeX (authors, journal, year, DOI) once it is published, and keep the ISMRM entry below it. -->
> **Paper coming soon:** *Spatial Patterns of White Matter Hyperintensities in Cerebral Border Zones and Their Evolution with Aging in Healthy Individuals.*
>
> Until it is out, please cite the ISMRM 2023 abstract:

```bibtex
@inproceedings{fathy2023catss,
  title     = {The Multi-Class Segmentation of the Human Cerebral Vasculature in {TOF-MRA}: A Supervised Deep Learning Approach},
  author    = {Fathy, Karim and Dumais, F{\'e}lix and C{\^o}t{\'e}, Samantha and Frederick, Blaise and Whittingstall, Kevin},
  booktitle = {Proceedings of the International Society for Magnetic Resonance in Medicine (ISMRM)},
  year      = {2023},
  note      = {Abstract 5271}
}
```

If you generate the CW labels with eICAB, please also cite it:

```bibtex
@article{dumais2022eicab,
  title   = {{eICAB}: A novel deep learning pipeline for Circle of Willis multiclass segmentation and analysis},
  author  = {Dumais, F{\'e}lix and Perez Caceres, Marco and Janelle, F{\'e}lix and Seifeldine, Kassem and Ar{\`e}s-Bruneau, No{\'e}mie and Gutierrez, Jose and Bocti, Christian and Whittingstall, Kevin},
  journal = {NeuroImage},
  volume  = {260},
  pages   = {119425},
  year    = {2022},
  doi     = {10.1016/j.neuroimage.2022.119425}
}
```

GitHub also offers a ready-made citation through the **"Cite this repository"** button in the sidebar.

## Acknowledgements

CATSS was developed at the Université de Sherbrooke (Whittingstall lab). It builds on [nnU-Net](https://github.com/MIC-DKFZ/nnUNet) (Isensee et al., *Nature Methods*, 2021) and [eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab) (Dumais et al., *NeuroImage*, 2022). The model was trained on TOF-MRAs from the CR-CHUS, MIDAS, Midnight Scan Club and OASIS-3 datasets.

This work was supported by NSERC (RGPIN-2024-06118), the Chaire de recherche en imagerie cérébro-vasculaire and the Fondation J-Louis Lévesque. Computations were run on the Narval supercomputer, operated by Calcul Québec and the Digital Research Alliance of Canada.

## License

<!-- TODO(karim): add a LICENSE file and name it here, e.g. "Released under the MIT License. See LICENSE." -->
License to be announced.
