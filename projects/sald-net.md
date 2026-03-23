---
layout: default
title: SALD-Net
permalink: /projects/sald-net/
---
# Summary
- This project presents a privacy-preserving 3D object detection framework for crowded hospital environments at the Medical Image Innovation Laboratory, Seoul National University, conducted during my master’s program and published in Signal, Image and Video Processing on September 22, 2025, as SALD-Net: Self-attention-integrated LiDAR-based 3D object detection network in a crowded hospital environment. 
- The study addressed severe occlusion, object overlap, sensor noise, and class imbalance that make robust perception difficult. 
- A hospital-specific Flash LiDAR point cloud dataset was constructed across four real clinical zones without relying on RGB imagery.
- Based on this dataset, SALD-Net was designed as a two-stage 3D detection architecture integrating point cloud preprocessing, class-balanced augmentation, and self-attention modules to jointly model local geometric features and global contextual dependencies.
- Experimental results demonstrated strong detection and instance separation performance in cluttered scenes, achieving a 3D mAP of 89.08% and showing clear gains over baseline methods, particularly for challenging object classes such as beds and wheelchairs.

---

# 1. Overview

**Autonomous mobile robots (AMRs) are increasingly deployed in hospitals** to mitigate workforce shortages through supporting tasks such as medication transport and patient guidance.

However, **accurate 3D object detection in hospital environments is significantly more challenging** than in outdoor autonomous driving scenarios due to dense human–object interactions, privacy constraints, and sensor limitations.

In this project, **SALD-Net, a self-attention integrated LiDAR-based 3D object detection network in a crowded hospital environment** is proposed.

---

# 2. Motivation
## 2-1. Why Hospital Detection Is Challenging

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig1.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 1. Illustration of challenges in 3D object detection in hospital environments. Background points are shown in black; object points and bounding boxes are color-coded by object class. (a) Occlusion: A person is partially occluded by a bed. (b) Overlap: Adjacent objects exhibit overlapping regions. (c) Combined: Both occlusion and overlap occur simultaneously. (d) Sparsity: Non-uniform sensor density leads to incomplete 3D representations</em></p>

#### Four key challenges due to unique domain characteristics of hospitals:

**1) Domain Gap**
The lack of domain-specific datasets and the presence of occluded or specialized objects (e.g., beds and wheelchairs) introduce significant detection difficulty.

**2) Sensor Noise & Low Resolution**
Flash LiDAR data are typically characterized by increased outliers and blurred object boundaries.

**3) Class Imbalance**
Wheelchairs and beds occur far less frequently than people, resulting in skewed training distributions.

**4) Occlusion & Overlapping Objects**
Dense multi-object motion due to move assistance makes instance separation highly challenging.

---

# 3. Proposed Approach

## 3.1 Self-Attention-Based Detection Architecture

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig2.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 2. The architecture of SALD-Net. The backbone extracts features via backbone-integrated self-attention mechanism (BAM). Foreground segmentation generates initial 3D box proposals, which are refined by the unified regional and grid (URG) RoI pooling head, enhanced with RoI feature-based self-attention mechanism (RAM). Fully connected (FC) layers output final confidence scores and bounding boxes</em></p>

SALD-Net is designed as a **two-stage end-to-end 3D object detector** tailored for **flash-LiDAR indoor environments**, where point clouds are sparse, low-resolution, and frequently occluded.

- **Stage 1**: Initial 3D proposal generation via PointNet++ backbone
- **Stage 2**: Proposal refinement using a Unified Regional and Grid (URG) RoI pooling head

### 3-1-1. Backbone-integrated self-attention mechanism (BAM)

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig3.png" style="max-width:100%;">
</p>
<p align="center"><em>Figure 3. Architecture details of the backbone-integrated self-attention mechanism (BAM)</em></p>

BAM models long-range relationships between point groups while preserving continuous geometry.

Given grouped features \( X = \{x_1, \dots, x_n\} \), global dependencies are captured by connecting all node pairs through a self-attention formulation applied prior to geometric discretization. Each node feature \( x_i \) represents a local point-wise grouped feature and \( n \) is the number of groups \( (i = 1, \dots, n) \). To capture global dependencies, all node pairs \( (x_i, x_j) \) are connected via an edge set \( E = \{r_{ij}, i,j = 1, \dots, n\} \), where each edge \( r_{ij} \) represents the interaction between the \( i^{\text{th}} \) and \( j^{\text{th}} \) nodes. These interaction terms are computed using a self-attention mechanism. 

In BAM, each node feature \( x_j \) is projected via linear layers into a key vector \( K_j \) and a value vector \( V_j \), while a query vector \( Q_i \) is derived from \( x_i \). The attention weight \( W_{ij} \) between the \( i^{\text{th}} \) and \( j^{\text{th}} \) point nodes is computed as \( W_{ij} = \text{softmax}(Q_i \cdot K_j^\top) \). The interaction term \( r_{ij} \) is obtained by multiplying \( W_{ij} \) and \( V_j \) as \( r_{ij} = W_{ij} \cdot V_j \). A global context-aware feature \( a_i = \sum_{j=1}^{n} r_{ij} \) is obtained and aggregated via multi-head attention, then concatenated with the local node feature \( x_i \) and upsampled through feature propagation. To address class imbalance between foreground and background regions, focal loss is applied. The architecture of BAM is illustrated in Figure 3.

This allows each local region to adaptively gather global structural cues without converting points into tokens or voxels:
- Geometry-aware context modeling
- Lightweight compared to full transformers

### 3-1-2. Unified Regional and Grid (URG) RoI Pooling head

Indoor medical objects (beds, wheelchairs, AMRs):
- vary greatly in scale,
- are partially occluded,
- contain limited LiDAR returns.

URG combines:
- **Regional RoI pooling** → captures surrounding context
- **Hierarchical grid pooling** → extracts multi-scale internal structure

This hybrid pooling strategy preserves both contextual surroundings and internal geometric structures, enabling robust detection under occlusion.

### 3-1-3. RoI feature-based self-attention mechanism (RAM)

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig4.png" style="max-width:100%;>
</p>
<p align="center"><em>Figure 4. Architecture details of the RoI feature-based self-attention mechanism (RAM)</em></p>

The RoI feature-based self-attention mechanism (RAM) refines the grouped point set from the hierarchical grid RoI pooling by integrating spatial and semantic relationships. 

The architecture of RAM is illustrated in Figure 4. The RAM takes as input the 3D coordinates $F_{\text{xyz}} \in \mathbb{R}^{N \times 3}$ and semantic features $F_f \in \mathbb{R}^{N \times C}$ of $N$ neighboring points. From $F_{\text{xyz}}$, a position embedding $p_e$ is computed via a fully connected (FC) layer. Semantic inputs $F_f$ are projected to key and value embeddings, $k_e$ via an FC layer, and $v_e$ via an MLP.

These coefficients update each embedding as follows: $v_e' = v_e + v_{\text{coef}} \odot p_e$, which incorporates spatial information into the value embedding; $p_e' = q_{\text{coef}} \odot p_e$, which adjusts the position embedding; $k_e' = k_{\text{coef}} \odot k_e$, which refines the semantic key embedding; and $pk_e' = qk_{\text{coef}} \odot pk_e$, which models their interaction. The attention map is computed as $\text{Attention map} = p_e' + k_e' + pk_e'$, which is then normalized via softmax and used to weight the updated value embedding $v_e'$ across $N$ neighboring points. The final aggregated feature is obtained as:
\begin{equation}
F_{\text{agg}} = \sum_{i=1}^{N} \text{Softmax}(\text{Attention map}^{(i)}) \odot v_e'^{(i)} \label{eq1}
\end{equation}

---

# 4. Implementation Details

## 4-1. Data Acquisition

The dataset was collected specifically for this study.
Due to hospital privacy regulations, it cannot be publicly released.
Access may be granted upon institutional agreement.

**LiDAR Sensor Configuration**

A total of 10,985 point cloud scenes were collected at the Veterans Health Service Medical Center between 2022 and 2023 using flash-type LiDAR sensors (NSL-1110AV, NANOSYSTEMS Corp., Gyeongsan, South Korea) operating at 5 frames per second (FPS).

LiDAR systems are broadly categorized into scanning and flash types. Unlike scanning LiDAR, which sequentially emits laser beams, flash LiDAR captures the entire scene simultaneously, providing robust and stable sensing for fixed indoor installations with simpler hardware and improved durability. 

Although flash LiDAR offers lower spatial resolution and a narrower field of view than scanning LiDAR, it is well suited for hospital environments, where reliable operation and privacy-preserving depth sensing are more important than long-range perception.

Each captured point cloud had a spatial resolution of 320 × 240 voxels with a sensing depth of up to 12 m.

**Sensor Deployment in a Hospital Environment**

A total of 19 LiDAR sensors were installed in fixed positions across four representative hospital zones: Radiology Department, Laboratory Medicine, Inpatient Ward A, and Inpatient Ward B.

These locations were deliberately selected to ensure:
- spatial diversity of indoor geometries
- varied human–robot interaction scenarios
- realistic collision-risk environments for AMR deployment.

This multi-zone configuration enabled the dataset to capture heterogeneous clinical workflows and dynamic object interactions, producing a representative benchmark for hospital navigation systems.

## 4-2. Data Preprocessing and Augmentation Process

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig5.png" style="max-width:100%;>
</p>
<p align="center"><em>Figure 5. Preprocessing steps for raw point cloud data. (a) Raw point cloud. (b) Voxel-based downsampling. (c) RANSAC-based filtering; red points indicate filtered wall points. (d) Final denoised point cloud after statistical outlier removal. Background points are shown in black; object points and bounding boxes are color-coded by object class</em></p>

To stabilize noisy hospital point clouds, a four-stage pipeline was designed:
- **Voxel-based downsampling for density normalization
Voxel-based downsampling was applied with a voxel size of 0.25 m:**
    - projects points into voxel grids,
    - replaces points within each voxel by their centroid,
    - reduces point density while maintaining global geometry.
      
- **RANSAC filtering to remove structural planes
To separate foreground objects from structural background, a RANSAC-based plane fitting algorithm was applied with:**
    - distance threshold: 0.2 m
    - minimum sampled points: 3
    - maximum iterations: 500
    
- **Statistical outlier removal for sensor noise reduction
To suppress measurement noise, statistical filtering was performed using:**
    - number of neighbors: 20
    - standard deviation ratio: 1.5

**Data Augmentation**
The preprocessed dataset of 10,985 scenes is split into 6,591 training, 2,197 validation, and 2,197 test samples. To mitigate class imbalance among robots, people, beds, and wheelchairs, GT sampling is adopted, a widely used data augmentation method originally introduced in SECOND.

Specifically, annotated objects from a database are inserted into other training scenes. This augmentation increases the number of robots, beds, and wheelchairs, as summarized in Table 1.

To avoid object overlaps, the z-coordinate of each inserted object is set to the lowest height among existing objects.
For horizontal positioning, inserted objects are placed beyond the largest existing x or y coordinates in the scene, depending on the spatial distribution of existing objects.

If existing objects are located along the positive x-axis, new objects are placed further along the y-axis, and vice versa.
Finally, scenes are normalized by centering the 3D coordinate system at (0, 0, 0), ensuring consistent spatial scaling across the dataset.

## 4-3. Software

**Framework**
- The proposed network and all comparative models were implemented using PyTorch 1.9.1.
- Baseline methods in Table 2 were reproduced using the OpenPCDet toolbox, a widely adopted open-source framework for 3D object detection.
- Except for adapting configurations to the hospital-specific dataset, all models retained their default settings provided in the official repository to ensure fair comparison.

**Training**
- Optimization was performed using the Adam optimizer.
- Batch size: 4
- Training epochs: 100
- A cosine annealing learning rate schedule was adopted:
  - Initial learning rate: 0.001
  - Increased gradually to a peak of 0.01 during the first 40% of training steps
  - Decreased smoothly to a minimal value over the remaining 60% of training.

- Online augmentation techniques included:
  - Random flipping
  - Random rotation in the range of −45° to 45°
  - Scaling with factors between 0.95 and 1.05

**Hardware**
- Training was conducted on an NVIDIA GeForce RTX 3090 GPU.
- The environment utilized CUDA 11.1 for GPU acceleration.
  
---

## 5. Results

SALD-Net significantly outperformed the baseline Part-A2 detector:

<p align="center">
  <img src="/images/projects/sald-net/sald-net_fig6.png" style="max-width:100%;">
</p>
<p align="center"><em>Fig 6. Qualitative comparison of 3D object detection results from baseline methods and SALD-Net on the test set. Background points are shown in black, and object points are color-coded by class. Top down BEV images provide overall scene context, while zoomed-in 3D RoIs highlight mispredicted objects. Detection errors—misclassified, missed, or over-detected—are indicated by arrows. Ground-truth and predicted objects are shown in red and aqua-blue bounding boxes, respectively</em></p>

- **3D mAP: 89.08%** 
- Overall improvement: **+19.56%** 
- Wheelchair detection: +22.85%p 
The model successfully separated tightly clustered objects that previous detectors failed to distinguish.

<p align="center">
  <img src="/images/projects/sald-net/sald-net_Table2.png" style="max-width:100%;>
</p>
<p align="center"><em>Table 2. Quantitative Performance comparison of 3D detection on our test dataset. The evaluation metrics are BEV AP(%), 3D AP(%) with an IoU threshold of 0.5 for robot, person, bed, and wheelchair classes, and inference speed measured in FPS</em></p>

<p align="center">
  <img src="/images/projects/sald-net/sald-net_Table3.png" style="max-width:100%;>
</p>
<p align="center"><em>Table 3. Ablation study results on the test set. Evaluation of the impact of data augmentation and self-attention mechanisms in different network modules. AUG: Applying data augmentation for the training set, BAM: backbone-integrated self-attention mechanism, RAM: RoI feature-based self-attention mechanism</em></p>

---

## 6. Technical Takeaways

This work demonstrates that:

- Indoor medical environments require **domain-specific perception modeling**
- Global relational reasoning is critical for dense human-object interaction scenes
- Dataset realism is as important as model architecture for safety-critical robotics

---

## 7. Future Work

Future extensions include:

- Multi-sensor fusion for improved spatial robustness
- Deployment-oriented optimization for real-time AMR navigation
- Transfer of the preprocessing pipeline to radar-based perception systems

