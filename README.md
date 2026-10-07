# CoastlineVLM-7B
## Learning Coastlines as Geometric Curves with Vision-Language Models

**[Rafia Malik](https://scholar.google.com/citations?user=14o8NMsAAAAJ)¹, [Bernhard Pfahringer](https://scholar.google.com/citations?user=PEv3OQUAAAAJ)¹, [Karin Bryan](https://scholar.google.com/citations?user=EbEzqL8AAAAJ)², [Mark Dickson](https://scholar.google.com/citations?user=Nbno3kwAAAAJ)², [Eibe Frank](https://scholar.google.com/citations?user=dUV_NvIAAAAJ)¹**  
¹ The University of Waikato, New Zealand  
² The University of Auckland, New Zealand

[![arXiv](https://img.shields.io/badge/arXiv-2606.10468-b31b1b.svg)](https://arxiv.org/abs/2606.10468)
[![Workshop](https://img.shields.io/badge/REO2-NeurIPS%202026-blue.svg)](#paper-and-publication-status)
[![Code](https://img.shields.io/badge/Code-Coming%20soon-lightgrey.svg)](#code-and-dataset)
[![Dataset](https://img.shields.io/badge/Dataset-Coming%20soon-lightgrey.svg)](#code-and-dataset)

A vision-language model that directly predicts an ordered coastline polyline, bypassing raster-to-vector post-processing.

## Updates

- 🎉 Short paper accepted at REO2, NeurIPS 2026.
- 📝 Extended manuscript under review at Neurocomputing.
  

## Overview

Coastlines are used as geometric curves in coastal monitoring, erosion assessment, and spatial analysis. However, most deep learning pipelines predict segmentation masks and recover the coastline through post-processing. **CoastlineVLM-7B** directly predicts an ordered coastline polyline, bypassing raster-to-vector post-processing.

Built on GeoChat-7B / LLaVA-1.5, it takes an aerial image and a task prompt and supports three tasks:

| Task | Model output |
| --- | --- |
| 🔍 **Coastline presence detection** | Whether a coastline is present in the image |
| 🏷️ **Geomorphic proxy classification** | Vegetation line, Cliff line, Gravel berm, Built structure line, or Waterline |
| 📍 **Coastline grounding** | An ordered sequence of coastline coordinate pairs |


### Framework overview

The diagram below summarizes the representation, model tasks, and key findings.

![CoastlineVLM-7B overview](assets/overview.png)


### Video overview

https://github.com/user-attachments/assets/b8b643fb-9234-4eb5-b678-b4232bce9083

## 💡 Key contributions

- **Direct coastline geometry:** Predicts ordered coastline polylines without an intermediate segmentation mask or raster-to-vector post-processing.
- **Coastline-Instruct:** An instruction-tuning dataset combining LINZ aerial imagery with coastline annotations from the New Zealand Coastal Change Dataset (NZCCD).
- **Geometric evaluation:** Assesses coastline localization using complementary tolerance-based and distance-based metrics.
- **Independent evaluation:** Tests zero-shot generalization on Australian Victorian Coastal Monitoring Program (VCMP) imagery.

## 🗂️ Coastline-Instruct dataset

Coastline-Instruct contains **17,977 aerial image tiles** with instructions for coastline presence, proxy classification, and polyline grounding.

| Property | Details |
| --- | --- |
| Image source | Land Information New Zealand (LINZ) |
| Coastline annotations | New Zealand Coastal Change Dataset (NZCCD) |
| Image size | 504 × 504 pixels |
| Geographic coverage | Nine New Zealand regions |
| Region Names | Auckland, Bay of Plenty, Gisborne, Hawke’s Bay, Northland, Otago, Taranaki, Waikato, West Coast |
| Images with a coastline | 8,988 |
| Images without a coastline | 8,989 |
| Training split | 16,221 images |
| Validation split | 881 images, Hawke's Bay |
| Test split | 875 images, West Coast |

Training, validation, and test regions are geographically separated. The West Coast region is held out as the test set to evaluate performance on unseen coastal environments.

| Input aerial image | Ground-truth coastline polyline |
| --- | --- |
| <img src="assets/coastline_instruct_raw.png" width="300"> | <img src="assets/coastline_instruct_label.png" width="300"> |

**Example instructions:**

- **Presence:** Is the coastline visible in this image? Answer only with Yes. or No.
- **Proxy type:** What coastline proxy defines the coastline in this image? Answer with one of: Vegetation line, Cliff line, Gravel berm, Built structure line, Waterline.
- **Grounding:** Give the coastline as a list of [x,y] points in normalized coordinates 0-100. Format exactly as `[[x1,y1],...,[xN,yN]]`.

## 📊 Results

### <img src="assets/nz.png" width="24" alt="New Zealand flag"> West Coast, New Zealand

Geometric coastline localization on the held-out West Coast test set. Higher is better for tolerance metrics (↑, %); lower is better for distance metrics (↓, m). Bold values indicate the best result in each column.

| Model | ≤5 m ↑ | ≤10 m ↑ | ≤20 m ↑ | Chamfer ↓ | Hausdorff ↓ | Mod. Avg. HD ↓ | EMD ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| U-Net | **69.60** | **76.80** | **85.40** | **9.43** | 37.74 | **11.99** | 21.12 |
| UNet++ | 65.60 | 72.00 | 79.50 | 13.78 | 68.25 | 19.67 | 28.59 |
| DeepLabV3+ | 6.60 | 9.20 | 10.60 | 18.02 | 78.41 | 32.26 | 38.93 |
| SegFormer | 18.40 | 20.60 | 23.60 | 17.99 | 74.48 | 30.97 | 39.61 |
| CoastlineVLM-7B | 33.30 | 60.80 | 85.20 | 11.98 | **31.84** | 13.42 | **17.32** |

U-Net provides stronger local boundary proximity. CoastlineVLM-7B achieves lower worst-case and global structural error, measured by Hausdorff distance and Earth Mover's Distance.

<p align="center">
  <img src="assets/west_coast_results.png" alt="West Coast qualitative comparison" width="90%">
</p>


Segmentation predictions can fragment or drift in difficult scenes, while CoastlineVLM-7B generally produces a more continuous coastline trace in these examples.

### <img src="assets/au.png" width="24" alt="Australian flag"> Cross-region zero-shot generalization to Australian data

Evaluation on **600 VCMP UAV patches from four Victorian sites**, with **no additional fine-tuning**.

| Model | ≤5 m ↑ | ≤10 m ↑ | ≤20 m ↑ | Chamfer ↓ | Hausdorff ↓ | Mod. Avg. HD ↓ | EMD ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| U-Net | 20.08 | 41.74 | 65.27 | 19.94 | 54.82 | 23.78 | 31.71 |
| CoastlineVLM-7B | **26.44** | **49.21** | **74.28** | **15.63** | **44.12** | **18.20** | **24.54** |

Both models show reduced performance relative to the New Zealand test set. CoastlineVLM-7B performs better across the reported metrics on the independent Australian dataset, supporting promising cross-region transfer.

<p align="center">
  <img src="assets/west_coast_results.png" alt="West Coast qualitative comparison" width="90%">
</p>


## 📦 Code and dataset availability

Code and Coastline-Instruct will be released after the Neurocomputing review process is complete. In the meantime, to request dataset access, email rm1050@students.waikato.ac.nz.

## 📖 Citation

If you use this work, please cite the arXiv manuscript:

```bibtex
@article{malik2026geometric,
  title   = {Geometric Coastline Localization using Vision-Language Models},
  author  = {Malik, Rafia and Pfahringer, Bernhard and Bryan, Karin and Dickson, Mark and Frank, Eibe},
  journal = {arXiv preprint arXiv:2606.10468},
  year    = {2026},
  url     = {https://arxiv.org/abs/2606.10468}
}
```

## Acknowledgements

<img src="assets/taiao_logo.png" width="180" alt="TAIAO logo">

This research is supported by the Time-Evolving Data Science and AI for Advanced Open Environmental Science (TAIAO) project.

We acknowledge LINZ and NZCCD for the imagery and coastline annotations used to develop Coastline-Instruct. We also thank the Victorian Coastal Monitoring Program (VCMP) team for providing the UAV orthomosaics used in the zero-shot cross-region evaluation.

