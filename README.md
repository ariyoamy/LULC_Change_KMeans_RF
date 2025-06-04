![banner](figures/combined_cluster_maps_panels.png)

# Evaluating Supervised and Unsupervised Land-Cover Classification in Urban Environments  
### A Sentinel-2 Case Study of Waltham Forest, London (2020–2024)

This repository compares a classical unsupervised K-means workflow with a supervised Random-Forest model for yearly land-cover mapping over Waltham Forest.  
The study assesses accuracy, temporal consistency and carbon footprint, demonstrating how choice of algorithm affects both performance and sustainability.

---

<details open>
<summary><strong>Table of Contents</strong></summary>

1. [Project motivation and background](#project-motivation-and-background)  
2. [Data source and preprocessing](#data-source-and-preprocessing)  
3. [Method overview](#method-overview)  
   * [Unsupervised K-means pipeline](#unsupervised-k-means-notebook-2)  
   * [Supervised Random-Forest pipeline](#supervised-random-forest-notebook-3)  
4. [Notebooks and quick start](#notebooks-and-quick-start)  
5. [Results](#results)  
6. [Environmental cost](#environmental-cost)  
7. [Walkthrough Video](#walkthrough-video)  
8. [References & Acknowledgements](#references--acknowledgements)

</details>



---

## Project Motivation and Background
Urban environments present some of the most spectrally complex scenes in satellite imagery. Buildings, roads, vegetation, water, and bare surfaces are frequently intermingled at the pixel level, making automated land-cover classification in cities a persistent challenge (Weng, 2012). The ability to monitor urban form and change over time is increasingly vital for climate adaptation, infrastructure planning, and environmental monitoring (UN-Habitat, 2020).

Traditionally, supervised machine learning approaches like Random Forests have been used for land-cover mapping, relying on labelled training samples to learn class distinctions. However, acquiring high-quality labelled data is time-consuming and subjective, especially in heterogeneous urban areas, where semantic boundaries are often unclear. Unsupervised methods such as K-means clustering offer a complementary alternative, classifying pixels based solely on their spectral characteristics. While typically less interpretable, they may detect subtle patterns or transitions missed by supervised models.

This project explores both approaches by applying a Random Forest and an unsupervised K-means algorithm to five years of Sentinel-2 imagery over the Borough of Waltham Forest in northeast London. The aim is to compare not only classification accuracy and spatial patterns, but also temporal consistency and environmental cost. In doing so, the project asks: how do supervised and unsupervised methods differ in their representation of urban land cover? And can clustering provide a more flexible, low-emission alternative in data-scarce settings?
<div align="center">
  <img src="figures/study_area.png" width="600"/>
  <p><em>Fig. 1: Study area - Waltham Forest, London</em></p>
</div>

---

## Data source and preprocessing  
This project uses Sentinel-2 Level-2A surface reflectance imagery from the Copernicus Open Access Hub to generate annual composites for land-cover classification over the London Borough of Waltham Forest.

The data was preprocessed as follows:
* **Time window**: April–August for each year (2020–2024).
These composites focus on the leaf-on season, when vegetation is fully active and spectral differences between land-cover types—such as vegetation, built surfaces, and water, are most pronounced. This improves model contrast and consistency.
* **Bands**: B2 (Blue), B3 (Green), B4 (Red), B8 (NIR), B11 (SWIR1), B12 (SWIR2), plus NDVI and NDBI
* **Cloud and shadow masking**: Pixels affected by clouds, cirrus, or shadows were removed using the Scene Classification Layer (SCL).  
* **Radiometric normalisation**: Pixel values in each band were clipped to the 2nd–98th percentile and scaled to [0, 1], using 2021 as a reference year. This ensures cross-year comparability in brightness and contrast. 
* **AOI**: Borough boundary of Waltham Forest (geoBoundaries ADM2).  

All processing steps are executed in 01_preprocessing.ipynb, exporting yearly 8-band 10 m GeoTIFFs to Google Drive.

---

## Method overview  
This project applies two contrasting machine learning approaches to classify land cover in Waltham Forest from Sentinel-2 imagery: unsupervised K-Means clustering and supervised Random Forest classification. Both methods follow a shared preprocessing pipeline but diverge in how they leverage label information.

<div align="center"> <img src="figures/unsupervised_pipeline.png" width="780"/> <p><em>Fig. 2.1: K-Means clustering pipeline (see Notebook 2).</em></p> </div>

### Unsupervised K-means  
K-Means clustering groups pixels based on spectral similarity without requiring any labelled data. It is particularly useful in exploratory settings or when class boundaries are subtle or subjective. After computing spectral features (including NDVI and NDBI), a random sample of 50,000 pixels from the 2021 composite is used to fit a StandardScaler and run K-Means with k = 4, selected using elbow and silhouette methods.

The four resulting clusters are semantically interpreted as:

* Cluster 0 – Dense urban (high SWIR)

* Cluster 1 – Vegetation (high NDVI)

* Cluster 2 – Light urban or residential (moderate SWIR)

* Cluster 3 – Open water (low NIR, high NDBI)

Clusters 0 and 2 are both considered “urban” under traditional classification but remain spectrally distinct here, offering granular insight into urban heterogeneity often collapsed in supervised models.

<div align="center"> <img src="figures/how_kmeans_works.png" width="520"/> <p><em>Fig. 2.2: Conceptual diagram of K-Means clustering (source: Medium, 2024).</em></p> </div>

### Supervised Random-Forest 
<div align="center"> <img src="figures/supervised_pipeline.png" width="780"/> <p><em>Fig. 2.3: Random Forest classification pipeline (see Notebook 3).</em></p> </div>
Random Forest (RF) is a supervised ensemble model trained on reference labels from ESA WorldCover 2021, with 250 points sampled per class. Each sample is enriched with 8 spectral features. The model is trained using an 80/20 train-test split, balancing classes and using 300 decision trees.

After training, the RF model is applied to annual composites from 2020 to 2024 to produce classified maps. Feature importance is also computed using SHAP values to interpret spectral contributions.

Evaluation against held-out test data from 2021 shows:

| Metric            | 2021 test |
|-------------------|-----------|
| Overall accuracy  | 0.81      |
| κ (kappa)         | 0.72       |
| F1 (urban)        | 0.77      |
| F1 (water)        | 0.93      |

Random Forest offers robust classification performance, particularly for stable land types like water. However, it tends to merge spectrally distinct subclasses (e.g. rooftops vs. roads), which K-Means retains.

---

## Notebooks and quick start  
| Notebook | Purpose |
|----------|---------|
| **01_preprocessing.ipynb** | Download, cloud-mask, radiometrically normalise and export annual composites. |
| **02_unsupervised_kmeans.ipynb** | K-means training, prediction and map generation. |
| **03_supervised_randomforest.ipynb** | Sample extraction, RF training, inference, feature importance and comparison plots. |

Clone the repo, open each notebook in Google Colab and run top-to-bottom.  

---

## Results  
| Figure | Description |
|--------|-------------|
| <img src="figures/kmeans_cluster_maps_panel.png" width="800"/> | K-means classification 2020–24 |
| <img src="figures/rf_classification.gif" width="300"/> | Random-Forest classification 2020–24 |
| <img src="figures/change_map.png" width="300"/> | Urban gain (red) and vegetation loss (yellow) 2020–2024 |
| <img src="figures/rf_feature_importance.png" width="300"/> | RF global feature importance, NIR and SWIR2 dominate |


Key insights:  
* **K-means captures urban spectral subtypes**, separating roads and rooftops, while RF merges them into a single "urban" class, offering finer-grained detail useful for urban planning or densification analysis.
* **Water area is highly stable** in both methods, with only **±0.15% change in RF** and **±2.13% in K-means**, indicating consistent performance for hydrological features.
* **Urban area change diverges sharply** between methods: RF detects a 14.1% increase, while K-means suggests a 7.8% decrease. This highlights a trade-off: RF offers label-aligned outputs, but K-means is more sensitive to spectral drift, making it better at revealing ambiguous or transitional land cover, though less stable for categorical comparisons.


---

## Environmental cost 
Two complementary approaches were used:

1. **Empirical tracking with CodeCarbon** – a decorator wraps the most energy-intensive cells (K-Means fit, RF training, full-image predictions).  
2. **Life-cycle discussion** – hardware manufacture, satellite upstream costs and hosting overheads are reviewed qualitatively.

### Measured emissions  

| Stage | Runtime (CPU) | Energy (kWh) | CO₂e (g) | Notes |
|-------|--------------:|-------------:|---------:|-------|
| Preprocessing | 6 min | 0.035 | 15.6 | tiled over ~1.5 × 10⁷ px |
| K-Means fit (50 k px, *k* = 4) | 2 min | 0.014 | 6.1 | single pass |
| RF training (1 000 pts × 4 classes) | 4 min | 0.028 | 12.4 | 300 trees, class-balanced |
| **Total** | **20 min** | **0.099** | **43.8** | Colab, europe-west4 |

*CodeCarbon v2.3.3 default UK grid intensity (≈ 443 g CO₂e kWh⁻¹).*  

A **43 g CO₂e** footprint is comparable to:

* boiling **0.6 kettles** of water  
* driving a petrol car **300 m**  
* streaming HD video for **6 minutes**

### Contextual impacts  

| Component | Impact channel | Mitigation in this study |
|-----------|----------------|--------------------------|
| **Compute** | Google Colab (T4 CPU only). Short runtimes keep energy use below 0.1 kWh. | No GPU required; region chosen for moderate grid mix. |
| **Data transfer / storage** | Five 80 MB GeoTIFFs stored in Drive. Negligible (< 0.001 kWh yr⁻¹). | Git LFS ignored for rasters; only notebooks and SVGs in repo. |
| **Hardware manufacture** | Embodied emissions of Colab servers and personal laptop. | Use of shared cloud nodes maximises utilisation; local laptop kept asleep during training. |
| **Satellite upstream** | Sentinel-2 launch (one-off ≈ 130 t CO₂e) amortised over > 10 million scene-equivalents. | AOI composites use < 0.00002 % of S-2 acquisitions: marginal impact. |

### Discussion  

* **Method choice matters:** the unsupervised pipeline emits ~40 % less than RF training-plus-inference yet still delivers useful change-detection insight.  
* **Scale sensitivity:** processing an entire GLA extent (610 km²) at 10 m would raise emissions by ≈ 6 × unless tiling is parallelised on low-carbon hardware.  
* **Greener defaults:** rerunning notebooks on Google’s carbon-aware `europe-north1` (Finland, ~75 g CO₂e kWh⁻¹) would cut the project footprint to **< 15 g CO₂e**.

---
## Walkthrough Video

A video walkthrough of the project is available here:
[Watch here](https://youtu.be/will_put_link_here_when_created)

---
## References & Acknowledgements
This repository was developed as a **final project** for the UCL undergraduate module:

> **GEOL0069: Artificial Intelligence for Earth Observation**  
> Department of Earth Sciences, University College London  
> Academic Year: 2024–2025  
> Special thanks to Dr. Michel Tsamados, as well as Weibin Chen and Connor Nelson, for the original notebook and teaching materials that formed the basis for this work.
[Professor Michel Tsamados](https://www.ucl.ac.uk/earth-sciences/people/academic/dr-michel-tsamados), [Weibin Chen](https://www.ucl.ac.uk/earth-sciences/people/research-students/weibin-chen) and [Connor Nelson](https://www.ucl.ac.uk/earth-sciences/people/research-students/connor-nelson)

- Tsamados, M., & Chen, W. (2022). *GEOL0069: Artificial Intelligence for Earth Observation – Course Notebook*. University College London. [https://cpomucl.github.io/GEOL0069-AI4EO/intro.html](https://cpomucl.github.io/GEOL0069-AI4EO/intro.html)  

- Raoof Naushad (2023). *Land Cover Classification using Sentinel-2 Dataset (Deep Learning)*  
  GitHub: [https://github.com/raoofnaushad/Land-Cover-Classification-using-Sentinel-2-Dataset](https://github.com/raoofnaushad/Land-Cover-Classification-using-Sentinel-2-Dataset)

- Tingzon, I., & Mahesh, A. (2024). *Land Use and Land Cover (LULC) Classification using Deep Learning* [Tutorial].  
  In Climate Change AI Summer School. Climate Change AI.  
  DOI: [10.5281/zenodo.11584954](https://doi.org/10.5281/zenodo.11584954)
  
- Bouza Heguerte, L., Bugeau, A., & Lannelongue, L. (2023). *How to estimate carbon footprint when training deep learning models? A guide and review*.  
  *Environmental Research Communications*.  
  DOI: [10.1088/2515-7620/acf81b](https://doi.org/10.1088/2515-7620/acf81b)  
  [hal.science/hal-04120582v2](https://hal.science/hal-04120582v2/document)

- Weng, Q. (2012). *Remote sensing of impervious surfaces in urban areas: Requirements, methods, and trends*.  https://doi.org/10.1016/j.rse.2011.02.030
    DOI: [https://doi.org/10.1016/j.rse.2011.02.030)  
  [[hal.science/hal-04120582v2](https://www.sciencedirect.com/science/article/pii/S0034425711002811)]([https://hal.science/hal-04120582v2/document](https://www.sciencedirect.com/science/article/pii/S0034425711002811))
     
- UN-Habitat. (2020). *World Cities Report 2020: The Value of Sustainable Urbanization*. United Nations Human Settlements Programme. https://unhabitat.org/sites/default/files/2020/10/wcr_2020_report.pdf

- Medium. (2024). *Understanding how K-Means Clustering Works (A detailed guide).  [https://unhabitat.org/sites/default/files/2020/10/wcr_2020_report.pdf](https://levelup.gitconnected.com/understanding-how-k-means-clustering-works-a-detailed-guide-9a2f8009a279)



---
