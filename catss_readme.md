# CATSS

### Cerebral Arterial Tree Segmentation Software

**Open-source, fully automated whole-brain segmentation and multi-class arterial labeling (ACA, MCA, PCA) from TOF-MRA.**

## Why CATSS?

Whole-brain arterial tree analysis on Time-of-Flight Magnetic Resonance Angiography (TOF-MRA) typically requires laborious manual tracing, especially when separating distal branches by vascular territory. **CATSS** automates the entire workflow end-to-end:

| Capability | Details | 
| ----- | ----- | 
| **Multi-Class Territorial Labeling** | Segments the visible distal arterial tree and labels branches into **ACA**, **MCA**, and **PCA** territories, rather than producing only a binary vessel mask. | 
| **Variant-Aware Classification** | Preserves true anatomical labels regardless of Circle of Willis variants (e.g., a PCA with a fetal origin retains its PCA label). | 
| **Near-Human Accuracy** | Achieves $73.1\% \pm 2.66\%$ whole-brain Dice on distal vessels, matching human expert intra-observer agreement ($71.5\%$). | 
| **Scanner & Artifact Robust** | Works across **1.5T, 3T, and 7T** scans, small or large FOVs, skull-stripped volumes, and images with Venetian blind artifacts. | 
| **Containerized Deployment** | Ships as a CUDA-accelerated Docker image with zero manual dependency wrangling. | 

## Pipeline

CATSS takes two co-registered NIfTI volumes per subject—a raw **TOF-MRA** (`*_TOF.nii.gz`) and a proximal **Circle of Willis segmentation** (`*_CW.nii.gz`)—generates a whole-brain arterial mask via **nnU-Net**, removes disconnected false positives, and propagates anatomical labels distally using the Circle of Willis mask as seed points.

```
flowchart LR
    A["TOF-MRA<br/>*_TOF.nii.gz"] --> C["nnU-Net<br/>Whole-Brain Vessel<br/>Segmentation"]
    B["Circle of Willis Mask<br/>*_CW.nii.gz<br/>(eICAB 1-18 grid)"] --> E["Seed-Guided<br/>Arterial Tree<br/>Labeling"]
    C --> D1["TOF_binary_segmentation_raw.nii.gz"]
    C --> D2["Post-Processing<br/>& Artifact Filtering"]
    D2 --> D3["TOF_binary_segmentation.nii.gz"]
    D2 --> E
    E --> F["TOF_multiclass_segmentation.nii.gz<br/>(ACA, MCA, PCA)"]
    F -.-> G["Downstream Applications<br/>e.g. Individualized Border Zone<br/>(BZR) & Territory Mapping"]

    style A fill:#1f2937,stroke:#60a5fa,color:#f9fafb
    style B fill:#1f2937,stroke:#60a5fa,color:#f9fafb
    style C fill:#0f4c81,stroke:#93c5fd,color:#ffffff
    style E fill:#0f4c81,stroke:#93c5fd,color:#ffffff
    style F fill:#065f46,stroke:#34d399,color:#ffffff
    style G fill:#1f2937,stroke:#9ca3af,stroke-dasharray: 5 5,color:#e5e7eb

```

## Validation & Benchmarks

CATSS was trained and evaluated on **126 multi-center 3T TOF-MRA scans** (`89` training, `27` testing, `10` validation) spanning four datasets: **OASIS-3** (65), **Institutional** (31), **MIDAS** (20), and the **Midnight Scan Club** (10). Training ground truth was built via [eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab) Circle of Willis initialization, Sato tubeness filtering, hysteresis thresholding, watershed anatomical tracing, and expert manual correction.

| Metric | Evaluation Set | Dice Score (Mean $\pm$ SD) | 
| ----- | ----- | ----- | 
| **Whole-Brain Arterial Tree** | Validation vs. Human Expert ($N = 10$) | $73.1\% \pm 2.66\%$ | 
| **Middle Cerebral Artery (MCA)** | Validation vs. Human Expert ($N = 10$) | $68.6\% \pm 3.5\%$ | 
| **Anterior Cerebral Artery (ACA)** | Validation vs. Human Expert ($N = 10$) | $67.9\% \pm 5.2\%$ | 
| **Posterior Cerebral Artery (PCA)** | Validation vs. Human Expert ($N = 10$) | $58.6\% \pm 4.7\%$ | 
| **Test-Retest Reproducibility** | Same-day repeat scans, Midnight Scan Club ($N = 10$) | $71.5\% \pm 2.71\%$ | 
| *Human Intra-Observer Ceiling* | *Repeat expert annotation (2-year interval)* | $71.5\%$ | 

> \[!NOTE\]
> Because distal intracranial vessels are thin tubular structures, even a 1–2 voxel boundary shift heavily penalizes Dice. Achieving $73.1\%$ against a $71.5\%$ human intra-observer baseline indicates CATSS operates near the practical human annotation ceiling.

## Quick Start

### 1. Prerequisites

* [Docker](https://docs.docker.com/get-docker/)

* **NVIDIA GPU (CUDA only):** Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

```
docker pull fetouh15/catss:latest

```

### 2. Prepare Your I/O Directory

CATSS reads inputs from and writes outputs to a **single I/O folder**.

> \[!CAUTION\]
> Each run expects **exactly one** `*_TOF.nii.gz` file and **exactly one** co-registered `*_CW.nii.gz` file inside the I/O directory.

```
io_dir/
├── sub-01_TOF.nii.gz
└── sub-01_CW.nii.gz

```

### 3. Run CATSS

**Docker (Local Workstation — GPU):**

```
docker run --rm --gpus all \
  -v /absolute/path/to/io_dir:/io \
  fetouh15/catss:latest

```

**Apptainer / Singularity (HPC Clusters):**

```
apptainer pull catss.sif docker://fetouh15/catss:latest
apptainer run --nv -B /absolute/path/to/io_dir:/io catss.sif

```

Adjust the container target path (:/io) if your Docker image mounts a different internal I/O directory.

### 4. Inspect Outputs

CATSS writes three NIfTI files back into your I/O directory:

| File | Description | 
| ----- | ----- | 
| `TOF_binary_segmentation_raw.nii.gz` | Raw nnU-Net binary segmentation prior to post-processing. | 
| `TOF_binary_segmentation.nii.gz` | Cleaned, post-processed whole-brain binary arterial tree. | 
| `TOF_multiclass_segmentation.nii.gz` | Final multi-class anatomically labeled arterial tree. | 

## Circle of Willis Labeling Scheme

CATSS follows the proximal Circle of Willis label grid defined by [**eICAB**](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab):

| Label ID (Left) | Label ID (Right) | Artery | Full Anatomical Name | 
| ----- | ----- | ----- | ----- | 
| `1` | `2` | **ICA** | Internal Carotid Artery | 
| `3` | `3` | **BAS** | Basilar Artery | 
| `4` | `4` | **AComm** | Anterior Communicating Artery | 
| `5` | `6` | **ACA-A1** | Anterior Cerebral Artery (A1 segment) | 
| `7` | `8` | **MCA-M1** | Middle Cerebral Artery (M1 segment) | 
| `9` | `10` | **PComm** | Posterior Communicating Artery | 
| `11` | `12` | **PCA-P1** | Posterior Cerebral Artery (P1 segment) | 
| `13` | `14` | **PCA-P2** | Posterior Cerebral Artery (P2 segment) | 
| `15` | `16` | **SCA** | Superior Cerebellar Artery | 
| `17` | `18` | **AChA** | Anterior Choroidal Artery | 

> \[!IMPORTANT\]
> Generating `*_CW.nii.gz` with [eICAB](https://gitlab.com/FelixDumais/vessel_segmentation_snaillab) is recommended, but any segmentation tool or manual mask works **as long as its integer IDs strictly match the table above**.

## Batch Processing

Running a full cohort? Loop over subject folders (each containing one `*_TOF.nii.gz` and one `*_CW.nii.gz`):

```
#!/usr/bin/env bash
set -euo pipefail

COHORT_DIR="/path/to/cohort"

for subj_dir in "${COHORT_DIR}"/sub-*; do
  [ -d "$subj_dir" ] || continue
  echo ">>> Running CATSS on $(basename "$subj_dir")"
  docker run --rm --gpus all \
    -v "${subj_dir}:/io" \
    fetouh15/catss:latest
done

```

### Quality Control Checklist

* **TOF Slab Coverage:** Verify that your TOF-MRA covers the full superior cerebral vasculature. Cropped clinical slabs omit distal cortical branches (and were the primary reason for scan exclusion in our OASIS-3 analysis).

* **Voxel-Grid Alignment:** `*_TOF.nii.gz` and `*_CW.nii.gz` must share the exact same image dimensions and affine header.

* **Motion Artifacts:** Severe head motion degrades distal vessel continuity; visually inspect `TOF_multiclass_segmentation.nii.gz` overlaid on your TOF MIP (e.g., in FSLeyes).

## Accompanying Study

CATSS was developed and validated in:

> **Spatial Patterns of White Matter Hyperintensities in Cerebral Border Zones and Their Evolution with Aging in Healthy Individuals**

Using CATSS-derived individual arterial trees in $N = 162$ cognitively unimpaired adults from OASIS-3 (mean age $70.7 \pm 8.42$ years, range 42.7–92.3), we constructed subject-specific **cerebral border zone region (BZR)** centerlines via Euclidean distance maps and showed that:

* $\sim 52\%$ of white matter hyperintensity (WMH) volume falls within $5\text{ mm}$ of the BZR centerline and $\sim 77\%$ within $10\text{ mm}$ (exceeding the $23.1\%$ and $46.0\%$ expected by chance).

* WMHs concentrate disproportionately within BZR in younger adults and become progressively more diffuse outside BZR with advancing age (Spearman $r = -0.47$, $p < 0.001$), independent of sex and cardiovascular risk factors.

## Citation

If you use **CATSS** in your research, please cite our manuscript:

```
@article{catss2026,
  title   = {Spatial Patterns of White Matter Hyperintensities in Cerebral Border Zones and Their Evolution with Aging in Healthy Individuals},
  author  = {[Author List]},
  journal = {[Journal Name]},
  year    = {2026},
  doi     = {[DOI]}
}

```

Please also cite the underlying frameworks:

* **nnU-Net:** Isensee F, Jaeger PF, Kohl SAA, Petersen J, Maier-Hein KH. *nnU-Net: a self-configuring method for deep learning-based biomedical image segmentation.* Nature Methods. 2021;18(2):203–211. [doi:10.1038/s41592-020-01008-z](https://doi.org/10.1038/s41592-020-01008-z)

* **eICAB:** Dumais F, Caceres MP, Janelle F, et al. *eICAB: A novel deep learning pipeline for Circle of Willis multiclass segmentation and analysis.* NeuroImage. 2022;260:119425. [doi:10.1016/j.neuroimage.2022.119425](https://doi.org/10.1016/j.neuroimage.2022.119425)

## Acknowledgments

Supported by the **Natural Sciences and Engineering Research Council of Canada** (NSERC Discovery Grant `RGPIN-2024-06118`), the **Chaire de recherche en imagerie cérébro-vasculaire**, and the **Fondation J-Louis Lévesque**. Computations were performed on **Narval**, managed by Calcul Québec and the Digital Research Alliance of Canada. Data used in training and validation were sourced from **OASIS-3**, **MIDAS**, and the **Midnight Scan Club**.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.