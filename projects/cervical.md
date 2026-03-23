---
layout: default
title: cervical
permalink: /projects/cervical/
---

# Summary
- This project presents a radiomics-based image quality screening framework for cervical image analysis at the Gachon AI Medical Device R&D Center, developed from July to November 2021 and published in the Journal of Biomedical Engineering Research on October 29, 2021, as A Radiomics-based Unread Cervical Imaging Classification Algorithm. 
- The study began from the observation that non-diagnostic cervical images significantly degraded the performance of an AI-based staging classification system.
- Based on prior studies and expert consultation, three screening criteria were defined: illumination inconsistency, image blur, and abnormal region-of-interest localization. A filtering framework was then designed using pixel intensity–based brightness, Laplacian-based blur estimation, and Euclidean distance–based center deviation to exclude diagnostically unsuitable images before model inference.
- All indicators showed statistical significance (p < 0.0001), and the framework automatically filtered 257 non-diagnostic images from 2,257 samples, improving ResNet-50 performance from 68.55% to 92.31% in F1-score and from 0.77 to 0.97 in AUC.

---

# 1. Overview

This study proposes a **radiomics-based pre-screening framework** to automatically identify and exclude unreadable medical images prior to deep learning model training.
Unlike conventional research that focuses on improving network architectures, this work demonstrates that **input data quality is a primary determinant of AI performance.**

By filtering diagnostically unreliable images before training, the framework reduces label noise and stabilizes feature learning, ultimately improving model accuracy and reliability in clinical environments.

---

# 2. Motivation

Real-world clinical datasets inevitably contain images degraded by:
- Motion artifacts
- Low contrast or acquisition errors
- Partial anatomical capture
- Severe noise or distortion

Such samples are often **difficult even for clinicians to interpret**, yet they remain in training datasets and introduce:
- Implicit label noise
- Feature ambiguity
- Unstable decision boundaries

Rather than increasing model complexity, this study explores a data-centric question:
**"Can AI performance improve simply by learning from diagnostically reliable data?"**

This perspective aligns with the shift toward **Data-Centric AI**, where dataset design becomes as critical as model architecture.

---

# 3. Proposed Approach

<p align="center">
  <img src="/images/projects/cervical/cervical_fig2.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 1. The flowchart illustration of the study method.</em></p>

Fig. 2 illustrates the overall workflow of the proposed method for automatically classifying unreadable cervical images.
The algorithm evaluates whether an image is diagnostically usable by quantifying brightness, blur, and anatomical positioning.
All metric values are normalized between **0 (0%) and 1 (100%)** using the full distribution of readable-image data.

## 3-1. Classification Indicators

Subtle morphological variations in cervical epithelial cells significantly influence diagnostic interpretation.
Therefore, pixel-level image quality differences can directly affect readability.
As shown in **Fig. 2**, three criteria are defined to classify unreadable images:

**1) Brightness**

The first indicator is the **average brightness value** of the image.
- Pixel intensities are summed and divided by the total pixel count to compute the mean brightness.
- Histograms are used to simultaneously visualize readable vs. unreadable datasets and compare their distributions.
- The acceptable brightness range is determined statistically using **t-test analysis and histogram-based thresholding.**
- Images with extreme brightness deviations that hinder visual interpretation are classified as unreadable.

**2) Blur**

The second indicator is the **degree of blur**, defined as the sharpness of the image.
- Blur is measured using the **Laplacian operator**, a second-order derivative widely used for edge detection.
- Lower values indicate blurred images; higher values indicate clearer images.
- Based on statistical analysis of readable images, the acceptable threshold is set within the **densely distributed upper 13% range of blur values.**
- This relative definition reflects the dataset-dependent nature of blur measurement.

**3) Region Adequacy (Cervical Position)**

The third indicator evaluates whether the cervical os region is properly captured.
- A template image with the cervical region centered is defined.
- Similarity between the template and an input image is computed using **Euclidean distance.**

Examples of unreadable cases identified using these criteria are presented in Fig. 3, including:
- excessively bright or dark images,
- blurred images,
- images where the cervical region is improperly captured.

<p align="center">
  <img src="/images/projects/cervical/cervical_fig3.png" style="max-width:50%;">
</p>
<p align="center"><em>Figure 2. Examples of Unread Cervical Imaging and Tagged Image, (a) Image of high brightness, (b) Image of low brightness, (c) blurred Image, (d) Image of inappropriately photographed cervical region, (e) Tagged Image of CAM misreading tag into cervical region, (f) Image of CAM reading cervical region right.</em></p>

## 3-2. Classification Procedure

<p align="center">
  <img src="/images/projects/cervical/cervical_fig4.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 3. The flowchart of Automated classification algorithm for Unread Cervical Imaging.</em></p>

The full decision process is depicted in **Figure. 3** and proceeds as follows:

**1) Input Stage**
A cervical image is provided to the algorithm.

**2) Threshold-Based Evaluation**
The image is tested against predefined thresholds for brightness, blur, and Euclidean-distance-based region adequacy.
Any image outside these ranges is classified as unreadable and excluded from model input.

**3) Cause-Oriented Categorization**
Each of the three indicators has a binary outcome (acceptable / unacceptable), producing:

2×2×2=8 unreadable-case combinations

The algorithm assigns each rejected image to one of these eight categories and outputs a message describing the specific cause.
Images satisfying all criteria are retained and recommended for expert diagnosis.

**Normalized Indicator Interpretation**

Within the normalized classification standard:
- Brightness: Values closer to 1 indicate higher intensity.
- Blur: Larger values indicate clearer images.
- Region (Euclidean Distance): Smaller values indicate better alignment of the cervical region.

---

# 4. Implementation Details

## 4-1. Data Acquisition

A total of 2,257 cervical images were collected using a cervical imaging diagnostic device (Dr. Cervicam, NTL Healthcare).
- The study protocol was approved by the Institutional Review Board (IRB No. EUMC 2015-10-004), and all data were fully anonymized prior to analysis.
- To ensure clear clinical separability, the dataset was composed using extreme stages of the cervical cancer progression spectrum, following the WHO classification system.

According to the WHO framework, cervical cancer develops through the following stages:

**Normal → Atypical → LSIL → HSIL → Invasive Cervical Cancer**

For this study:
- **Readable Images (2,000 total)**
  - 1,000 Normal cases (control group)
  - 1,000 HSIL cases (lesion group)
- **Unreadable Images (257 total)**
  - Images judged by experts as diagnostically unusable.

All images were stored as **24-bit JPG files** and resized to **256 × 256 pixels** before model processing.

## 4-2. Data Preprocessing and Augmentation Process
Unreadable images in the collected dataset were initially labeled by clinical experts due to issues such as abnormal brightness, reflections, or other visual defects that prevented cervical identification.

**Resolution and Aspect Ratio Normalization**
The raw dataset contained heterogeneous image sizes (2048×1536, 1280×960, 1504×1000) and varying aspect ratios.

To standardize spatial representation:
- The shorter side of each image was used as the reference dimension.
- Images were center-cropped to produce a consistent square format.

**Removal of Non-Anatomical Tags**

Some images contained metadata tags in the lower-right corner.
These tags could be mistakenly interpreted as anatomical structures by AI models.

<p align="center">
  <img src="/images/projects/cervical/cervical_fig1.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 4. Network Structure used in analyzing Tag, (a) CNN Structure, (b) CAM Structure.</em></p>

Therefore:
- Tag regions were removed prior to analysis to prevent false feature learning.

**Center Alignment of the Cervical Region**

The diagnostically important cervical os was not always located at the image center.
- The main anatomical region was spatially normalized,
- Feature calculations reflected anatomical characteristics rather than positional bias.

## 4-3. Software

**Framework**
- The proposed network and all comparative models were implemented using TensorFlow/Keras, which was the most widely used framework for transfer learning with ResNet50 in medical imaging research.
- Standalone Keras (v2.4) was already being phased into tf.keras, making TensorFlow-integred Keras the more likely setup.

**Training**
- Optimization was performed using the Adam optimizer.
- Batch size: 16
- Training epochs: 300
- A cosine annealing learning rate schedule was adopted:
  - Initial learning rate: 0.0001

**Hardware**
- The environment utilized CUDA 10.1
  
---

# 5. Results

The proposed filtering algorithm significantly improved the overall classification performance by removing unreadable images before model training.

<p align="center">
  <img src="/images/projects/cervical/cervical_table1.png" style="max-width:100%;">
</p>
<p align="center"><em>Table 1. Table of independent sample test results for pixel values in Unread Cervical Imaging and Read Cervical Imaging.</em></p>

- Using the entire dataset (without filtering): Precision 72.64%, Recall 65.04%, F1-score 68.55%, Accuracy 78.88%
- Using the refined dataset (after applying the proposed algorithm): Precision 91.59%, Recall 93.09%, F1-score 92.31%, Accuracy 91.61%

After filtering, the original 2,000 readable images were further refined to 1,357 high-quality samples, while the unfiltered dataset (1,257 images including unreadable cases) was used as the comparison baseline.

<p align="center">
  <img src="/images/projects/cervical/cervical_fig5.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 5. Histogram of the image pixel distribution of Read Cervical Imaging and Unread Cervical Imaging, (a) Brightness,
(b) Blur, (c) Euclidian distance.</em></p>

<p align="center">
  <img src="/images/projects/cervical/cervical_fig6.png" style="max-width:60%;">
</p>
<p align="center"><em>Figure 6. Comparison of image pixel values of Unread Cervical Imaging and Read Cervical Imaging.</em></p>

Receiver Operating Characteristic (ROC) analysis also demonstrated clear performance gains:

- AUC = 0.77 for the unfiltered dataset
- AUC = 0.97 for the filtered dataset

The difference between the two ROC curves was statistically significant (t-test, P < 0.0001), confirming that removing unreadable images leads to substantially improved diagnostic classification performance.

<p align="center">
  <img src="/images/projects/cervical/cervical_table2.png" style="max-width:100%;">
</p>
<p align="center"><em>Table 2. Comparative Table of Performance Evaluation of ResNet-50 according to Exclusion Unread Cervical Imaging Algorithm.</em></p>

<p align="center">
  <img src="/images/projects/cervical/cervical_fig7.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 7. The results ROC analysis of ResNet-50 according to Exclusion Unread Cervical Imaging Algorithm.</em></p>

---

# 6. Technical Takeaways

This work demonstrates that:
## 6-1. Data-Centric AI Can Rival Model-Centric Improvements
Performance gains were achieved without changing the backbone network, emphasizing that dataset reliability is a first-order factor in AI success.

## 6-2. Image Readability Can Be Quantified
Radiomics enables translation of subjective clinical judgments into:
- Reproducible numerical metrics
- Automated dataset validation
- Scalable quality control pipelines

## 6-3. Practical Deployment Value

This framework is immediately applicable to:
- Pre-training dataset validation
- Multi-center data harmonization
- Clinical AI safety pipelines

---

# 7. Future Work

## 7-1. Quality-Aware Training (Soft Weighting Instead of Hard Filtering)
Incorporate readability scores directly into loss weighting.

## 7-2. Artifact Localization Models
Move from detecting whether an image is degraded to where degradation occurs.

## 7-3. Automated Data-Curation Pipelines
Continuous dataset refinement during model lifecycle (active data governance).

## 7-4. Multi-Modality Extension
Apply the framework to CT, MRI, ultrasound, and CBCT environments.
