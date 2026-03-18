# Smartphone-Based 3D Intraoral Scanning with iPhone TrueDepth Camera

## Problem Statement

This project presents my Bachelor thesis completed in 2022 at the [ETH Zurich Computer Graphics Laboratory (CGL)](https://cgl.ethz.ch/research/medical/). The thesis explores whether smartphones can serve as low-cost intraoral scanners by leveraging the Face ID hardware of an iPhone XS Max.

The work was conducted in the context of a broader research effort at CGL on **Burden-Reduced Cleft Lip and Palate Care and Healing**. In that project, one major goal was to reconstruct the 3D geometry of intra-oral anatomy from image-based inputs in order to support the generation of orthopedic appliances via 3D printing. More information about the larger research project is available [here](https://brc.ch/research/cleft-lip-and-palate/).

Smartphones are much more accessible than dedicated medical scanning devices and tend to depreciate quickly, making used hardware affordable. This thesis therefore investigated whether commodity depth sensing hardware could become a practical option for selected 3D scanning tasks in medicine.

## Overview of the Project

The project consists of two main components:

### 1. Native iOS app for RGB-D capture
A custom iPhone app was developed in Swift to capture synchronized RGB-D videos using the **TrueDepth camera system** of the iPhone. The app records raw color and depth frames together with camera calibration data, enabling accurate downstream 3D reconstruction.

### 2. Python-based 3D reconstruction pipeline
A Python pipeline reconstructs textured 3D meshes from the captured RGB-D sequences. It estimates camera poses, fuses multiple frames into a unified surface representation, extracts a triangle mesh, and improves the visual quality of the final texture.

The method was evaluated on several types of scans, including:
- two clinical infant cases:
- a 4-month-old baby with bilateral cleft lip and 
- a 1-year-old baby with isolated palate cleft.
- upper jaw and palate scans,
- plaster casts,


## Project Methods


## Hardware

This project uses an Apple **iPhone XS Max** and its **TrueDepth camera system** as a low-cost RGB-D sensing device for intraoral scanning.

<p align="center">
  <img src="images/iPhoneXsTrueDepthCamera.png" alt="iPhone XS Max TrueDepth camera system" width="24%" />
  <img src="images/dot_projector_infraredcamera.PNG" alt="Infrared view of the TrueDepth dot pattern" width="29%" />
  <img src="images/dot_projector_infraredcamera.PNG" alt="Intraoral mirror used for capture" width="29%" />

</p>

The TrueDepth system combines an **infrared camera**, **dot projector**, and **infrared illuminator** to project a structured light pattern onto the scene and recover depth from its deformation.

Because the sensor operates only beyond a minimum distance of roughly **15 cm**, direct capture of the palate and other hard-to-reach intraoral regions is limited. To address this, the setup uses an **intraoral mirror**, which improves visibility of the palate and otherwise occluded regions while preserving sufficiently accurate depth measurements in the reflected view.


## Native Swift iPhone App

The iOS app was developed to record and export RGB-D scans using the iPhone TrueDepth camera. It extends Apple’s depth streaming functionality into a workflow tailored to 3D reconstruction.

### Key features
- **Capture screen** for recording and save raw synchronized RGB and depth frames
- **Gallery / scan browser** for reviewing captured scans
- **Export workflow** to share a captured scan and camera calibration as a ZIP archive, for example via AirDrop to a Mac

<p align="center">
  <img src="images/iPhoneScreenshotApp1.PNG" alt="Capture screen" width="31%" />
  <img src="images/iPhoneScreenshotApp2.jpg" alt="Gallery screen" width="31%" />
  <img src="images/iPhoneScreenshotApp3.jpg" alt="Export workflow" width="31%" />
</p>



## 3D Reconstruction Pipeline in Python

The reconstruction pipeline was designed with a strong focus on geometric accuracy.

### 1. RGB-D capture
The iOS app records synchronized color and depth frames and saves calibration data for each scan.

### 2. Image rectification and preprocessing
Color and depth images are rectified using the camera calibration and lens distortion information.

### 3. Point cloud generation
Each RGB-D frame is converted into a 3D point cloud using the camera intrinsics. Surface normals are estimated to support robust frame-to-frame registration.

### 4. Region extraction for intraoral scans (optional)
For mouth-focused scans, facial landmarks [1] are used to localize the mouth region in the RGB images. This region is then further refined with point cloud processing and clustering to isolate the relevant intraoral geometry.

### 5. Pairwise registration
Consecutive frames are aligned use Point-to-Plane variant of the Iterative Closest Point algorithm (ICP) for registration on slightly downsampled point clouds.

### 6. Camera pose estimation
Relative frame transformations are accumulated and globally optimized using pose graph optimization [2] to recover a consistent camera trajectory with high geometric accuracy.

### 7. Volumetric fusion and mesh extraction
The aligned RGB-D frames are integrated into a scalable Truncated Signed Distance Field (TSDF) volume by applying Marching Cube Algorithm [3]. A triangle mesh is then extracted from the volume.

### 8. Texture enhancement
A separate color map optimization [4] stage improves the appearance of the reconstructed surface by sharpening and refining texture information.

### 9. Quantitative evaluation
The final mesh is aligned to a ground-truth scan generated by a high-end intraoral scanner and evaluated by rigidly aligning it to a ground-truth intraoral scan and computing nearest-neighbour point-to-point distances, summarized by the mean and standard deviation in millimetres.

## Results

The project produced **textured 3D reconstructions from smartphone RGB-D input** and demonstrated that consumer hardware can achieve surprisingly strong geometric performance in selected intraoral scanning scenarios.


### Upper jaw scan adult

<table>
  <tr>
    <td align="center" width="33%">
      <img src="images/adult-upperjaw_iPhoneReconstruction.png" alt="Upper jaw scan reconstructed with the iPhone with texture" width="100%"><br>
      <sub><b>(b)</b> iPhone RGB-D reconstruction with optimized texture.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/adult-upperjaw_iPhoneReconstructionNoTexture.png" alt="Upper jaw scan reconstructed with the iPhone without texture" width="100%"><br>
      <sub><b>(c)</b> Same iPhone reconstruction without texture to better show the recovered geometry.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/adult-upperjawIntraOralCamera.png" alt="Upper jaw scan reconstructed with an intraoral camera" width="100%"><br>
      <sub><b>(a)</b> Ground-truth upper jaw scan captured with an intraoral camera.</sub>
    </td>
  </tr>
</table>

#### Quantitative Evaluation

For each point of the iPhone reconstruction, the nearest-neighbour distance to the aligned ground-truth scan is computed and visualized directly on the surface using the colour bins below.

<table>
  <thead>
    <tr>
      <th>Point distance</th>
      <th>Mapped colour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&lt; 0.5 mm</td>
      <td><span style="color: rgb(117,230,0);">●</span> light green</td>
    </tr>
    <tr>
      <td>&lt; 1.0 mm</td>
      <td><span style="color: rgb(0,166,0);">●</span> dark green</td>
    </tr>
    <tr>
      <td>&lt; 2.0 mm</td>
      <td><span style="color: rgb(242,230,0);">●</span> yellow</td>
    </tr>
    <tr>
      <td>&lt; 3.0 mm</td>
      <td><span style="color: rgb(204,97,0);">●</span> orange</td>
    </tr>
    <tr>
      <td>≥ 3.0 mm</td>
      <td><span style="color: rgb(204,0,0);">●</span> red</td>
    </tr>
  </tbody>
</table>


<table>
  <tr>
    <td align="center" width="50%">
      <img src="images/upperjawErrorMapBottom.png" alt="Upper jaw error map bottom view" width="100%"><br>
      <sub><b>(a)</b> Error map of the iPhone reconstruction, bottom view.</sub>
    </td>
    <td align="center" width="50%">
      <img src="images/upperjawErrorMapTop.png" alt="Upper jaw error map top view" width="100%"><br>
      <sub><b>(b)</b> Error map of the iPhone reconstruction, top view.</sub>
    </td>
  </tr>
</table>

Most of the higher error values are located in the teeth regions, while the smoother palate area is reconstructed much more accurately.





### Baby bilateral cleft lip

<table>
  <tr>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipReconstruction.png" alt="Bilateral cleft lip reconstruction with texture" width="100%"><br>
      <sub><b>(a)</b> iPhone RGB-D reconstruction with texture of a 4-month-old baby with bilateral cleft lip.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipReconstructionNoTexture.png" alt="Bilateral cleft lip reconstruction without texture" width="100%"><br>
      <sub><b>(b)</b> The same reconstruction without texture.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipReconstructionIntraOralCamera.png" alt="Bilateral cleft lip ground truth from intraoral camera" width="100%"><br>
      <sub><b>(c)</b> Ground-truth scan acquired with an intraoral camera.</sub>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipReconstructionIntraOralCamera.png" alt="Bilateral cleft lip intraoral camera ground truth" width="100%"><br>
      <sub><b>(a)</b> Ground truth from the intraoral camera.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipErrorTop.png" alt="Bilateral cleft lip error map with ground truth" width="100%"><br>
      <sub><b>(b)</b> Error map overlaid with the ground truth.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/bilateralcleftlipErrorOnlyTop.png" alt="Bilateral cleft lip error map only" width="100%"><br>
      <sub><b>(c)</b> Error map alone.</sub>
    </td>
  </tr>
</table>


### Baby isolated palate cleft

<table>
  <tr>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftReconstruction.png" alt="Isolated palate cleft reconstruction with texture" width="100%"><br>
      <sub><b>(a)</b> iPhone RGB-D reconstruction with texture of a 1-year-old baby with isolated palate cleft.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftReconstructionNoTexture.png" alt="Isolated palate cleft reconstruction without texture" width="100%"><br>
      <sub><b>(b)</b> The same reconstruction without texture.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftReconstructionIntraOralCamera.png" alt="Isolated palate cleft ground truth from intraoral camera" width="100%"><br>
      <sub><b>(c)</b> Ground-truth scan acquired with an intraoral camera.</sub>
    </td>
  </tr>
</table>



<table>
  <tr>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftReconstructionIntraOralCamera.png" alt="Isolated palate cleft intraoral camera ground truth" width="100%"><br>
      <sub><b>(a)</b> Ground truth from the intraoral camera.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftError.png" alt="Isolated palate cleft error map with ground truth" width="100%"><br>
      <sub><b>(b)</b> Error map overlaid with the ground truth.</sub>
    </td>
    <td align="center" width="33%">
      <img src="images/isolatedpalatecleftErrorOnly.png" alt="Isolated palate cleft error map only" width="100%"><br>
      <sub><b>(c)</b> Error map alone.</sub>
    </td>
  </tr>
</table>



### Summary Quantitative outcomes
<table>
  <thead>
    <tr>
      <th colspan="4">Measured errors to ground truth in <b>mm</b> with iPhone RGB-D reconstruction</th>
    </tr>
    <tr>
      <th></th>
      <th>Upper jaw</th>
      <th>Palate</th>
      <th>Plaster cast teeth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Mean</b></td>
      <td>0.46</td>
      <td>0.24</td>
      <td>0.47</td>
    </tr>
    <tr>
      <td><b>Std</b></td>
      <td>0.52</td>
      <td>0.20</td>
      <td>0.46</td>
    </tr>
  </tbody>
</table>

<br>

<table>
  <thead>
    <tr>
      <th colspan="3">Measured errors to ground truth in <b>mm</b> with iPhone RGB-D reconstruction (clinical infant cases)</th>
    </tr>
    <tr>
      <th></th>
      <th>Baby bilateral cleft</th>
      <th>Baby isolated palate cleft</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Mean</b></td>
      <td>1.09</td>
      <td>0.77</td>
    </tr>
    <tr>
      <td><b>Std</b></td>
      <td>0.92</td>
      <td>0.76</td>
    </tr>
  </tbody>
</table>

<br>



### Interpretation
These results show that smartphone-based RGB-D scanning can be sufficiently accurate for certain reconstruction tasks, especially on smoother anatomical regions such as the palate. Accuracy tends to decrease in more difficult regions, such as teeth boundaries, occluded areas, and narrow border regions.


## Limitations
The iPhone TrueDepth camera only works reliably within a limited distance range (>=15cm), which makes close intraoral capture difficult. Border regions and occluded surfaces, such as tooth edges or deeper cavities, are harder to observe consistently and can lead to missing geometry or higher reconstruction error. 
Overall, dedicated intraoral scanners still provide better precision, easier access to narrow anatomical regions, and more reliable performance in clinical settings.


## Thesis Context

**Title of the thesis:** *Smartphone-based intraoral scanning*  
**Institution:** ETH Zurich, Computer Graphics Laboratory (CGL)  
**Year:** 2022

This repository presents the project in condensed form for portfolio and hiring purposes. The full thesis, additional details, and implementation material can be shared on request.

## Contact

For more details and access please contact: ![Contact email](images/contact-email.png)

### References

[1] Vahid Kazemi and Josephine Sullivan. *One Millisecond Face Alignment with an Ensemble of Regression Trees*. Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2014.

[2] Sungjoon Choi, Qian-Yi Zhou, and Vladlen Koltun. *Robust Reconstruction of Indoor Scenes*. Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2015.

[3] William E. Lorensen and Harvey E. Cline. *Marching Cubes: A High Resolution 3D Surface Construction Algorithm*. ACM SIGGRAPH Computer Graphics, 21(4):163–169, 1987.

[4] Qian-Yi Zhou and Vladlen Koltun. *Color Map Optimization for 3D Reconstruction with Consumer Depth Cameras*. ACM Transactions on Graphics (TOG), 33(4), 2014.

