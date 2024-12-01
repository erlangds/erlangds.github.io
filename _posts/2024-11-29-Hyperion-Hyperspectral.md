---
#image: /assets/img/subsidence background.jpg # or base64 URI
title: "Supervised Hyperion Clay Minerals Mapping using Random Forest and Support Vector Machine"
tag: [GIS, Machine Learning, Hyperspectral]
date: 2024-08-05 00:00:00 +0800
categories: [GIS]
description: Hyperspectral clay study in Sidoarjo Mud Volcano
layout: post
---
Universitas Gadjah Mada, Bachelor Thesis, 2024
## Background
Sidoarjo mud has been closely studied since the 2006 eruption that led to continuous hot mud flow until today. For over 18 years, the impacts of this disaster have significantly affected the surrounding areas, impacting both the environment and the economy. Sidoarjo is known for its clay minerals rich in rare earth elements (REE). The presence of REEs in these clay minerals has triggered scientific interest due to their potential applications in high-tech industries such as electronics, renewable energy, and advanced materials development.

<!-- Add script to the <head> of your page to load the embeddable map component -->
<script type="module" src="https://js.arcgis.com/embeddable-components/4.31/arcgis-embeddable-components.esm.js"></script>
<!-- Add custom element to <body> of your page -->
 <arcgis-embedded-map style="height:400px;width:700px;" item-id="0f37274e809049e5a63ae210839a4d70" theme="dark" portal-url="https://lugdc.maps.arcgis.com" share-enabled></arcgis-embedded-map>

In mineral mapping, the use of multispectral images such as ASTER and LANDSAT is very common in Indonesia, employing VNIR-SWIR color composites or band ratios. However, multispectral imaging has limitations in the number of bands and the spectral range it can cover, especially in the SWIR range where distinctive absorption features of clay minerals are typically found. To address these challenges, hyperspectral imaging offers a solution with a significantly larger number of bands and narrower spectral ranges.

## Data and Method
### Data
This project uses Hyperion EO-1 hyperspectral data collected from U SGS Earth Explorer  with the acquisition time in 2015. This data was downloaded in L1R format, which means Level 1 Radiometrically Corrected. Additionally, XRD data were analyzed from the field.

### Method
Unlike multispectral data, hyperspectral data are typically processed using <span style="color: yellow;">dimension reduction methods</span>. Kruse (2003) proposed a method named "Hourglass" that includes several processing techniques such as Minimum Noise Fraction (MNF), Pixel Purity Index (PPI), n-D Visualizer, and then mapped spatially with Random Forest algorithm.

> ***The script was run in anaconda environment by using Jupyter notebook, the code can be found at the end of this storymap*** 

This method aims to identify the purest group of pixel characteristics (endmembers) from the spectra by analyzing their shapes and absorption features, and comparing them with spectra from the USGS Spectral Library at the pixel level.

## MNF
The analysis begins with MNF (minimum noise fraction) conversion using ENVI software, with input data from atmospheric correction results. During this process, 163 Hyperion image bands are simplified to the best bands, with initial bands having the least noise and later bands having the most.

![MNF](https://erlangds.github.io/assets/img/Hyperion/mnf.jpg){: lqip="/assets/img/Hyperion/mnf.jpg" }{: w="300" h="100" }{: .center }_MNF results (the first 12 bands)_

Spatially, the MNF transformation results show that the initial MNF bands (1-8) clearly depict the Sidoarjo mud puddle with minimal noise. In contrast, the later MNF bands (9-12) have high noise levels, making it difficult to recognize the mud puddle. This trend continues for bands beyond 12. Therefore, the MNF transformation effectively filters the 163 bands down to just 8 bands with the best image quality and least noise.

## PPI and N-D visualizer
Pure pixels (PPI) are extracted at this stage and visualized in n-dimensional space, revealing a dense distribution of pixels. These dense pixels share the same spectral characteristics and pixel values, allowing them to be delineated and identified as endmembers. There are 8 endmembers in the study area.

![PPI](https://erlangds.github.io/assets/img/Hyperion/AI-08.png){: lqip="/assets/img/Hyperion/AI-08.png" }{: .center }_Left: N-D visualizer, illustrating the distribution of pixels with similar values (endmember)., Right: these pixels mapped spatially on the Lapindo mud volcano_

## Spectra Matching (Training data)
Since Random Forest is a supervised classification method, the input data must be labeled. Therefore, the Spectral Analyst tool is used to identify the mineral types to be used as labels by matching the endmember spectra with reference spectra from the  USGS . For example, endmember 2 has three water absorption regions and an Al-OH metal hydroxyl absorption at a wavelength of 2203 nm, which matches the spectral characteristics of the montmorillonite.

![spectra](https://erlangds.github.io/assets/img/Hyperion/montmorilonit.jpg){: lqip="/assets/img/Hyperion/montmorilonit.jpg" }{: .center }_Endmember 2 has a similar spectrum to montmorillonite_

The same process was applied to all 8 endmember spectra. Then, all pixel data with labels were trained and mapped using the Random Forest Algorithm.

## Results
The resulting maps appear to make geological sense, showing that the mineral distribution is spread in a circular pattern. The composition is predominantly a mixture of kaolinite and smectite, covering 46.37% of the study area.

<div class="juxtapose" style="max-width: 450px; margin: auto;">
    <img src="https://erlangds.github.io/assets/img/Hyperion/SIdoarjo_polos.jpg" style="width: 100%;" />
    <img src="https://erlangds.github.io/assets/img/Hyperion/SIdoarjo.jpg" style="width: 100%;" />
</div>


The Random Forest algorithm successfully classified the Hyperion hyperspectral image with an overall accuracy (OA) of 99.4% and a kappa score of 0.993, slightly higher than the SVM algorithm's 98.9% OA and 0.987 kappa score. The source of error came from the misclassification of the mineral montmorillonite. The H2O absorption regions and the Al-OH metal hydroxyl absorption played the most significant roles in model creation.

<div class="juxtapose" style="max-width: 450px; margin: auto;">
    <img src="https://erlangds.github.io/assets/img/Hyperion/RF.png" style="width: 100%;" />
    <img src="https://erlangds.github.io/assets/img/Hyperion/SVM.png" style="width: 100%;" />
</div>

<script src="https://cdn.knightlab.com/libs/juxtapose/latest/js/juxtapose.min.js"></script>
<link rel="stylesheet" href="https://cdn.knightlab.com/libs/juxtapose/latest/css/juxtapose.css">