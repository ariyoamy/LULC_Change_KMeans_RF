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

* Cluster 0 – Urban (residential/roads) (moderate SWIR, low NDVI)

* Cluster 1 – Vegetation (high NDVI, moderate SWIR)

* Cluster 2 – Industrial / Light roofs (high SWIR, low NDVI)

* Cluster 3 – Open water (low NIR, very low SWIR)

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
| <img src="figures/kmeans_cluster_maps.gif" width="300"/> | K-means classification 2020–24 |
| <img src="figures/rf_classification.gif" width="300"/> | Random-Forest classification 2020–24 |
| <img src="figures/rf_kmeans_change_maps.png" width="800"/> | Urban gain (red) and vegetation loss (yellow) 2020–2024 |
| <img src="figures/rf_feature_importance.png" width="300"/> | RF global feature importance, NIR and SWIR2 dominate |

### Land-Cover Area Comparison (%), 2024

| Class        | Random Forest (%) | K-Means (%) | Notes |
|--------------|-------------------|-------------|-------|
| **Urban**    | 39.43              | 48.48       | RF underestimates urban compared to K-Means |
| **Industrial** | 0.00              | 6.16        | RF merges this into general urban — K-Means separates it |
| **Vegetation** | 53.65            | 39.39       | RF likely over-assigns vegetation |
| **Water**    | 6.92               | 5.97        | High agreement between methods |

**Key observations:**
- **K-Means identifies an "Industrial" subtype** (~6%) which RF collapses into the broader urban category.
- **Vegetation is overestimated in RF**, likely due to shadows or mixed pixels in semi-urban green areas.
- **Water class shows excellent consistency**, suggesting both methods reliably detect hydrological features.
- These differences highlight a core trade-off: **RF aligns better with labelled categories**, while **K-Means reveals finer spectral distinctions**, valuable in heterogeneous urban environments.



Key insights:  
* **K-means captures urban spectral subtypes**: Unlike the supervised RF model, K-means detected a fourth cluster—interpreted as **industrial or light-roofed buildings** (e.g., schools, depots, warehouses). These areas exhibit distinct spectral properties (e.g. lower NIR, higher SWIR) compared to dense residential urban zones. While this added detail provides **granular insight into urban heterogeneity**, it comes at the cost of lacking semantic labels, requiring post hoc interpretation via visual inspection or auxiliary datasets like Google Earth.
* **Water area is highly stable** in both methods, with only **±0.15% change in RF** and **±2.13% in K-means**, indicating consistent performance for hydrological features.
* **Urban area change diverges sharply** between methods: RF detects a 14.1% increase, while K-means suggests a 7.8% decrease. This highlights a trade-off: RF offers label-aligned outputs, but K-means is more sensitive to spectral drift, making it better at revealing ambiguous or transitional land cover, though less stable for categorical comparisons.


---

## Environmental cost 
This project incorporates environmental accountability by tracking the computational energy usage and estimated carbon emissions associated with each stage of the workflow. While the emissions are minimal in absolute terms, the broader goal is to cultivate sustainable habits in spatial computing and remote sensing research.

### Measured emissions  

| Stage           | Runtime (hrs) | Energy (kWh) |  CO₂e (g) |    Cost (£) | Notes                   |
| --------------- | ------------: | -----------: | --------: | ----------: | ----------------------- |
| Preprocessing   |        0.0280 |     0.000560 |     0.131 |      0.0002 | GEE export + rescaling  |
| K-Means (k = 4) |        0.1145 |     0.002289 |     0.533 |      0.0007 | Cluster fit + inference |
| Random Forest   |        0.0646 |     0.001292 |     0.301 |      0.0004 | Train, predict, compare |
| **Total**       |    **0.2071** | **0.004141** | **0.965** | **£0.0013** | All stages, CPU only    |


*Assumptions: 20 W CPU, 0.233 kg CO₂/kWh (UK grid), £0.30/kWh.*

At just under **1 g CO₂e**, the entire analysis emitted less than:

- **1 minute of HD video streaming**
- **Boiling ⅒ of a kettle**
- **Driving ~5 meters in a petrol car**



### Contextual impacts  

| **Component**                   | **Impact Channel**                                                                                              | **Mitigation or Rationale**                                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Compute (Colab CPU)**         | Used Google Colab with virtualised CPU-only runtime (no GPU). Default region likely europe-west4 (Netherlands). | No local compute needed. Short runtimes (~12 mins total) and low power use (20 W CPU) kept energy below 0.005 kWh.  |
| **Cloud infrastructure**        | Google data centres are carbon-neutral since 2007 and aim for 24/7 carbon-free energy by 2030.                  | Shared infrastructure improves efficiency. Carbon-aware scheduling supports cleaner regions like europe-north1.      |
| **Satellite data (Sentinel-2)** | Launch emissions ~130 t CO₂ per satellite amortised across >10 million global acquisitions.                    | AOI use < 0.00002% of S-2 capacity. Promotes efficient reuse of open-access datasets.                                |
| **Storage & transfer**          | Less than 1 GB of data stored on Google Drive during processing. Files deleted after use.                       | Lifecycle energy < 0.001 kWh/yr. No raster storage in GitHub. Only notebooks and vector assets included.             |
| **Code efficiency**             | Avoided unnecessary recomputation, and GPU use. Workflows designed to be single-pass and modular.        | Fast execution, reproducibility, and minimal overhead. No fine-tuning or iterative ML models required.               |
| **Hardware manufacturing**      | Embodied emissions from shared Colab hardware and personal laptop.                                              | Laptop remained idle during runs. Cloud execution leverages already-active servers, maximising hardware utilisation. |

### Discussion and Mitigation  

* Scale matters: Extending to a city-scale project (e.g., all of Greater London) could increase emissions by 6× or more, depending on resolution and tiling.

* Colab region choice: Using carbon-aware Colab Pro with europe-north1 or us-west1 could cut emissions by up to 80%.

* Reusable code: Modular notebooks and reproducible pipelines reduce the need to re-run experiments, improving computational efficiency per insight.

* Unsupervised methods like K-Means offer lower-carbon alternatives in scenarios where labelled data is unavailable or overfitting is a risk.

Ultimately, while this project’s carbon footprint is scientifically negligible, its methodical accounting reflects a responsible approach to computational geography. As machine learning expands in environmental domains, even small efficiencies can add up to meaningful impact.

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

- Google. (2024). 2024 Environmental Report. https://blog.google/outreach-initiatives/sustainability/2024-environmental-report/

- Google. Innovating sustainable ideas. Growing renewable solutions. https://datacenters.google/operating-sustainably/

- Strubell, E., Ganesh, A., & McCallum, A. (2020). Energy and Policy Considerations for Modern Deep Learning Research DOI: https://doi.org/10.1609/aaai.v34i09.7123
  https://ojs.aaai.org/index.php/AAAI/article/view/7123



---
