---
# For reference on model card metadata, see the spec: https://github.com/huggingface/hub-docs/blob/main/modelcard.md?plain=1
# Doc / guide: https://huggingface.co/docs/hub/model-cards
---

# Model Card for Copernicus-FM

Copernicus-FM (Copernicus Foundation Model) is a unified foundation model for Earth observation, pretrained on the Copernicus-Pretrain dataset (18.7M aligned Sentinel images). It can process any spectral or non-spectral sensor modality through extended dynamic hypernetworks and flexible metadata encoding, and it was evaluated on the Copernicus-Bench suite of 15 downstream tasks covering Sentinel-1, Sentinel-2, Sentinel-3 and Sentinel-5P.

## Model Details

### Model Description

Copernicus-FM is an extension of the DOFA foundation model able to process any spectral or non-spectral sensor modality using extended dynamic hypernetworks and flexible metadata encoding. The model is pretrained on the Copernicus-Pretrain dataset with masked image modeling (MIM) and continual distillation.

- **Developed by:** Yi Wang, Zhitong Xiong, Chenying Liu, Adam J. Stewart, Thomas Dujardin, Nikolaos Ioannis Bountos, Angelos Zavras, Franziska Gerken, Ioannis Papoutsis, Laura Leal-Taixé, and Xiao Xiang Zhu — Technical University of Munich (TUM), Munich Center for Machine Learning (MCML), National Technical University of Athens (NTUA), National Observatory of Athens (NOA), Harokopio University of Athens, and NVIDIA.
- **Funded by [optional]:** Not reported by the authors.
- **Shared by [optional]:** Model weights, the Copernicus-Pretrain dataset, the Copernicus-Bench datasets, and the Copernicus-Embed-025deg embedding dataset are publicly shared on Hugging Face (`wangyi111/Copernicus-FM`, `wangyi111/Copernicus-Pretrain`, `wangyi111/Copernicus-Bench`, `wangyi111/Copernicus-Embed-025deg`).
- **Model type:** Unified multimodal Earth-observation foundation model: a Vision Transformer (ViT) encoder whose patch-embedding convolution kernels are generated dynamically by sensor-aware hypernetworks (spectral hypernetwork for sensors with a spectral response; variable hypernetwork for non-spectral variables), with metadata (geolocation, patch area, time) integrated as Fourier encodings added to the patch tokens.
- **Language(s) (NLP):** Not applicable — the model processes satellite imagery, not natural language. Non-spectral input variable names are encoded offline into 128-d vectors with a frozen Llama 3.2 encoder (language-guided embeddings) before being passed to the variable hypernetwork.
- **License:** Model weights: CC-BY-4.0. Code: Apache License 2.0, with portions of third-party code under MIT and CC-BY-NC-4.0 (see LICENSE and NOTICE in the official repository). Datasets (Copernicus-Pretrain and the newly curated Copernicus-Bench datasets): CC-BY-4.0. Sourced benchmark datasets retain their own licenses (CC0-1.0, MIT, CDLA-Permissive-1.0, CC-BY-4.0).
- **Finetuned from model [optional]:** Pretrained from scratch on Copernicus-Pretrain with MAE-style masked image modeling plus continual distillation from frozen teachers (DINOv2 for Sentinel-2-derived RGB, SoftCon for Sentinel-1/2). The architecture extends the DOFA dynamic-hypernetwork design and the MAE encoder-decoder structure.

### Model Sources [optional]

- **Repository:** https://github.com/zhu-xlab/Copernicus-FM
- **Paper [optional]:** Wang et al., "Towards a Unified Copernicus Foundation Model for Earth Vision", ICCV 2025 (oral), arXiv:2503.11849 — v1 14 Mar 2025, v2 29 Mar 2025, v3 31 Jul 2025 (latest).
- **Demo [optional]:** Official demo notebook `Copernicus-FM/demo.ipynb` (usage examples for spectral and non-spectral inputs): https://github.com/zhu-xlab/Copernicus-FM/blob/main/Copernicus-FM/demo.ipynb

## Uses

### Direct Use

The model is intended for image feature extraction from satellite observations: given an image patch from any sensor modality (with its spectral response or variable name, and optionally metadata), it produces dense feature maps suitable as a frozen or fine-tuned backbone for downstream tasks such as land use/land cover classification and segmentation, cloud detection, flood change detection, local climate zone classification, biomass regression, and air quality (NO2, O3) regression. The paper demonstrates out-of-the-box use via k-NN classification and linear probing on frozen encoders, without task-specific fine-tuning of the backbone.

### Downstream Use [optional]

Copernicus-FM encoders can be fine-tuned or used with task heads for the 15 hierarchical tasks of Copernicus-Bench: (L1, preprocessing) cloud detection on Sentinel-2 and Sentinel-3 OLCI; (L2, base applications) land use/land cover classification (EuroSAT-S1/S2, BigEarthNet-S1/S2, LC100Cls-S3, LCZ-S2) and segmentation (DFC2020-S1/S2, LC100Seg-S3); (L3, specialized applications) flood change detection (Flood-S1, Kuro Siwo), biomass regression (Biomass-S3), and air quality regression (AQ-NO2-S5P, AQ-O3-S5P). The model is also intended as a provider of grid embeddings for weather/climate research (e.g., the Copernicus-Embed-025deg global embedding map aligned with ERA5). The authors also plan to contribute the model to open-source libraries such as TorchGeo.

### Out-of-Scope Use

- Non-Sentinel satellite missions: the model and its pretraining data are restricted to the Sentinel series (S1, S2, S3, S5P) plus Copernicus DEM; generalization to other satellite constellations is out of scope for this release (stated limitation).
- Temporal range of about one year: pretraining data are anchored around the year 2021; behavior on substantially different time periods is not validated by the authors.
- Applications requiring native multimodal fusion or time-series processing within the model: the authors identify these as future work.
- Any operational decision-making (e.g., flood response, air quality alerts) without domain validation and complementing ground truth data.

## Bias, Risks, and Limitations

The paper does not report a dedicated fairness/sociotechnical risk analysis; the points below are derived from what the paper and the official repository do report about data coverage, sensor resolution, evaluation scope, and stated limitations.

**Biases.**
- Geographic sampling bias: Sentinel-1/2 pretraining patches are sampled with a Gaussian strategy (standard deviation 50 km) around the top-10K populated cities, plus 40K polar locations; densely populated regions are therefore over-represented relative to uninhabited land and oceans (land-only coverage with a 0.5° coastal buffer).
- Sensor coverage bias: the model is pretrained and evaluated only on Sentinel series sensors (S1 SAR, S2/S3 optical, S5P atmospheric columns) and Copernicus DEM; it is not designed for other sensors.
- Temporal bias: all pretraining data are anchored around 2021 (~1 year of acquisition), so long-term dynamics, trends, and rare events outside that window are under-represented.
- Task/label bias: benchmark label provenance varies — 9 datasets are derived from existing sources (CloudSEN12+, EuroSAT, EuroSAT-SAR, BigEarthNet v2.0, DFC2020, Kuro Siwo, So2Sat LCZ42) and 6 are newly curated by the authors (Cloud-S3, LC100Cls-S3, LC100Seg-S3, Biomass-S3, AQ-NO2-S5P, AQ-O3-S5P); label quality and class definitions differ across tasks.
- Metadata availability bias: several benchmark datasets lack geolocation and/or time metadata (DFC2020-S1/S2, LCZ-S2), which limits the model's metadata-driven advantage on those tasks.

**Risks.**
- Downstream users may over-estimate model performance on regions, seasons, land cover types, or sensors under-represented in Copernicus-Pretrain and Copernicus-Bench.
- Model outputs for regression tasks (biomass, air quality) are estimates that should be validated against in-situ measurements before being used in environmental monitoring or policy contexts.
- The authors note that the benefit of the S1/2 distillation component decreases with longer training times and larger models, so the pretrained checkpoint may not be optimal for every downstream recipe.
- The patch-area metadata is approximate by construction: due to geographical projection it does not strictly reflect surface area (stated by the authors as accurate enough for pretraining).
- Training the model carries a substantial compute cost (one pretraining run = 512 NVIDIA A100 GPU-hours).

**Limitations (stated by the authors in the latest paper version, arXiv:2503.11849v3).**
- "As a limitation, the scope of this work is restricted to the Sentinel series and a temporal range of about 1 year. Future work will thus also explore efficient scaling for satellite expansion, as well as native model support for multimodal fusion and time series processing."

### Recommendations

- Users (both direct and downstream) should be made aware of the risks, biases and limitations listed above, in particular the Sentinel-only, single-anchor-year scope of the pretraining data.
- Before deployment, validate model performance on region-, season-, and land-cover-specific test data relevant to the application domain, rather than relying solely on the Copernicus-Bench aggregate scores.
- For regression products (biomass, NO2, O3), complement predictions with ground-based measurements and report uncertainty.
- When using downstream classification/segmentation heads, follow the benchmark protocol (linear probing with SGD for classification; UPerNet decoder with AdamW for segmentation/regression; 50 epochs) for comparable results.
- Comply with the licenses of the model weights and datasets (CC-BY-4.0), and of each sourced benchmark dataset (CloudSEN12+ CC0-1.0, EuroSAT MIT, EuroSAT-SAR CC-BY-4.0, BigEarthNet v2.0 CDLA-Permissive-1.0, Kuro Siwo MIT, So2Sat LCZ42 CC-BY-4.0, DFC2020 CC-BY-4.0).

## How to Get Started with the Model

Use the code below to get started with the model (adapted from the official repository):

```python
import torch
from src.model_vit import vit_base_patch16  # or vit_large_patch16 for the ViT-L checkpoint

# create model
model = vit_base_patch16(num_classes=10, global_pool=False)

# load pre-trained weights
path = './weights/CopernicusFM_ViT_base_varlang_e100.pth'  # 558 MB; large variant: CopernicusFM_ViT_large_varlang_e100.pth (1.5 GB)
check_point = torch.load(path)
state_dict = check_point['model'] if 'model' in check_point else check_point
msg = model.load_state_dict(state_dict, strict=False)
print(msg)

# encode a spectral image (any stack of spectral bands)
img = torch.randn(1, 4, 224, 224)
meta = torch.full((1, 4), float('nan'))  # [lon, lat, delta_time, patch_token_area], assume unknown
wvs = [490, 560, 665, 842]   # wavelength: B,G,R,NIR (Sentinel 2)
bws = [65, 35, 30, 115]      # bandwidth: B,G,R,NIR (Sentinel 2)
language_embed = None        # N/A for spectral input
kernel_size = 16
input_mode = 'spectral'
logit, embed = model(img, meta, wvs, bws, language_embed, input_mode, kernel_size)
print(logit.shape, embed.shape)  # embed contains the encoded image features
```

For non-spectral inputs (e.g., Sentinel-5P NO2/CO/SO2/O3 or Copernicus DEM), load the pre-encoded variable-name embeddings (`varname_embed_llama3.2_1B.pt`, 128-d per variable, encoded with Llama 3.2-1B) and use `input_mode='variable'` with the appropriate patch size (4 for S5P, 64 for DEM). See the official demo notebook for detailed examples: https://github.com/zhu-xlab/Copernicus-FM/blob/main/Copernicus-FM/demo.ipynb

## Training Details

### Training Data

Copernicus-FM is pretrained on **Copernicus-Pretrain**, a unified pretraining dataset of aligned imagery from all major operational Copernicus Sentinel missions, organized in ~310K regional grids of 0.25°×0.25° (consistent with ERA5 coordinates), densely covering the whole land surface and near-land ocean. Images were downloaded with Google Earth Engine around the anchor year **2021** and quality-checked (corrupt or NaN-dominated images removed). The final union subset contains 312,567 grid cells and 18,713,054 images; the all-modality-aligned ("joint") subset used for pretraining contains 219,543 grid cells and 15,720,353 images. The dataset is available in GeoTIFF and WebDataset formats on Hugging Face (`wangyi111/Copernicus-Pretrain`, total file size 20.7 TB), licensed CC-BY-4.0.

| Modality | Type | GSD | Image size | # Grid cells | # Patches | # Timestamps | # Total images |
|---|---|---|---|---|---|---|---|
| Sentinel-1 GRD | SAR | 10 m | 264×264×2 | 247,723 | 1,067,267 | ~4 | 4,227,387 |
| Sentinel-2 TOA | MS | 10 m | 264×264×13 | 247,723 | 1,067,267 | ~4 | 4,218,065 |
| Sentinel-3 OLCI | MS | 300 m | 96×96×21 | 281,375 | 281,375 | ~8 | 2,189,561 |
| Sentinel-5P CO | atmos. | 1 km | 28×28 | 306,097 | 306,097 | 1–12 | 2,104,735 |
| Sentinel-5P NO2 | atmos. | 1 km | 28×28 | 291,449 | 291,449 | 1–12 | 1,752,558 |
| Sentinel-5P SO2 | atmos. | 1 km | 28×28 | 262,259 | 262,259 | 1–12 | 1,366,452 |
| Sentinel-5P O3 | atmos. | 1 km | 28×28 | 306,218 | 306,218 | 1–12 | 2,556,631 |
| Copernicus DEM (GLO-30) | elevation | 30 m | 960×960 | 297,665 | 297,665 | 1 | 297,665 |
| **Copernicus-Pretrain (union)** | | | | **312,567** | **3,879,597** | | **18,713,054** |

On average, each grid cell contains 4 local S1/S2 time-series pairs (16 images each), 8 S3 images, 7/6/5/8 S5P CO/NO2/SO2/O3 images, and 1 DEM image. S1/S2 locations were sampled with a Gaussian strategy (std 50 km) around the top-10K populated cities (~1M locations) plus 40K polar locations; each location contributes 4 seasonal 264×264 patches. S3 tiles with bright-pixel percentage above 20% were filtered as cloudy; fully-NaN S5P tiles were removed; S5P products are 12 monthly means per variable; the DEM provides a single elevation image per grid.

### Training Procedure

#### Preprocessing [optional]

Raw dataset converted to WebDataset format (one grid cell = one minimum sample per shard). Per iteration, one image is sampled from each of the eight modalities of one grid cell. Normalization: S1/S2 channel-wise mean/std; S3 multiplied by the per-channel Google Earth Engine `COPERNICUS_S3_OLCI` scale factor; S5P raw values with NaN pixels replaced by zero (monthly means use the first day of the month); DEM divided by 10000. Metadata per image: geolocation (central lon/lat), patch area (from GSD × patch size, in km², adapted dynamically to cropping), and time (days since 1970-01-01; DEM uses the first day of 2015).

#### Training Hyperparameters

- **Backbones released:** ViT-B/16 (embed_dim 768, 12 blocks, 12 heads, patch 16) and ViT-L/16 (embed_dim 1024, 24 blocks, 16 heads, patch 16); decoder embed_dim 512, 8 blocks, 16 heads.
- **Training regime:** AdamW optimizer, weight decay 0.05; base learning rate 1.5e-4 (defined at batch 256, linearly scaled), effective batch size 288; warmup 10 epochs followed by cosine decay; 100 epochs on the 220K-grid joint subset; MAE-style masked image modeling with masking ratio 0.7 for all modalities; metadata drop probability 0.7.
- **Distillation:** continual distillation from frozen teachers — DINOv2 (ViT-B/14) for Sentinel-2-derived RGB channels (loss weight 0.1) and SoftCon (ViT-B/14) for Sentinel-1/2 full channels (loss weight 0.2), cosine-similarity loss after global average pooling with a student projection head.
- **Augmentations:** random resized cropping (scale [0.2, 1.0]) and random horizontal flipping.
- **Patch sizes per modality:** 16×16 for S1/S2, 8×8 for S3, 4×4 for S5P, 64×64 for DEM (kernel dynamically resized from the 16×16 base).
- **Variable encoding:** non-spectral variable names encoded with Llama 3.2 (frozen) into 128-d vectors (zero additional cost at training time).
- **Fourier encoding ranges:** wavelengths [1e2, 1e9] nm; bandwidths [1, 1e9] nm; lon/lat [1e-4, 720]; patch area [1e-3, 5.1e8] km²; time [1, 365.25] days.

#### Speeds, Sizes, Times [optional]

- **Compute:** one training run takes 512 GPU hours on NVIDIA A100 GPUs, i.e., 128 node hours on one compute node with 4× A100 (40 GB); the official README states that training the ViT-B/16 model for 100 epochs on the full 220K-grid dataset takes ~60 hours on 8× A100 (40 GB).
- **Released checkpoints:** `CopernicusFM_ViT_base_varlang_e100.pth` (558 MB) and `CopernicusFM_ViT_large_varlang_e100.pth` (1.5 GB), plus `varname_embed/varname_embed_llama3.2_1B.pt` (language embeddings for S5P/DEM variable names); an older DOFA-S location checkpoint is kept in a `deprecated/` folder.
- **Pretraining model components (not all released):** encoder (above), MAE decoder (embed_dim 512, 8 blocks, 16 heads), dynamic decoder patch predictors, and distillation projection head.
- **Dataset size:** Copernicus-Pretrain total file size 20.7 TB (Hugging Face), raw GeoTIFF and streaming WebDataset formats.

## Evaluation

### Testing Data, Factors & Metrics

#### Testing Data

Evaluation is performed on **Copernicus-Bench**, a suite of 15 downstream datasets in three application levels, spanning S1, S2, S3 and S5P (9 datasets derived from existing sources with permissive licenses, 6 newly curated by the authors, all available on Hugging Face `wangyi111/Copernicus-Bench`):

| Level | Name | Task | Train/Val/Test images | Image size | # Classes | Source | License |
|---|---|---|---|---|---|---|---|
| L1 | Cloud-S2 | segmentation (cloud) | 1699/567/551 | 512×512×13 | 4 | CloudSEN12+ | CC0-1.0 |
| L1 | Cloud-S3 | segmentation (cloud) | 1197/399/399 | 256×256×21 | 5 | new | CC-BY-4.0 |
| L2 | EuroSAT-S1 | classification (LULC) | 16200/5400/5400 | 64×64×2 | 10 | EuroSAT-SAR | CC-BY-4.0 |
| L2 | EuroSAT-S2 | classification (LULC) | 16200/5400/5400 | 64×64×13 | 10 | EuroSAT | MIT |
| L2 | BigEarthNet-S1 | classification (LULC) | 11894/6117/5991 | 120×120×2 | 19 | BigEarthNet v2.0 | CDLA-Permissive-1.0 |
| L2 | BigEarthNet-S2 | classification (LULC) | 11894/6117/5991 | 120×120×12 | 19 | BigEarthNet v2.0 | CDLA-Permissive-1.0 |
| L2 | LC100Cls-S3 | classification (LULC) | 5181/1727/1727* | 96×96×21 | 23 | new | CC-BY-4.0 |
| L2 | DFC2020-S1 | segmentation (LULC) | 3156/986/986 | 256×256×2 | 10 | DFC2020 | CC-BY-4.0 |
| L2 | DFC2020-S2 | segmentation (LULC) | 3156/986/986 | 256×256×13 | 10 | DFC2020 | CC-BY-4.0 |
| L2 | LC100Seg-S3 | segmentation (LULC) | 5181/1727/1727* | 96×96×21 (label 288×288) | 23 | new | CC-BY-4.0 |
| L3 | Flood-S1 | change detection (flood) | 3000/1000/1000* | 224×224×2 | 3 | Kuro Siwo | MIT |
| L3 | LCZ-S2 | classification (local climate zone) | 15000/5000/5000 | 32×32×10 | 17 | So2Sat LCZ42 | CC-BY-4.0 |
| L3 | Biomass-S3 | regression (biomass) | 3000/1000/1000* | 96×96×21 (label 288×288) | – | new | CC-BY-4.0 |
| L3 | AQ-NO2-S5P | regression (air quality) | 1480/493/494* | 56×56×1 | – | new | CC-BY-4.0 |
| L3 | AQ-O3-S5P | regression (air quality) | 1480/493/494* | 56×56×1 | – | new | CC-BY-4.0 |

*Default mode is 1 image for segmentation and 2 for change detection; LC100Cls-S3/LC100Seg-S3/Biomass-S3/AQ-S5P also support seasonal (4-image) modes. Ablations use k-NN on EuroSAT and linear probing on LC100Cls-S3; the climate grid-embedding study uses 10K globally sampled grids with ERA5 targets.

#### Factors

Because Copernicus-FM processes physical sensor data rather than people, evaluation factors are adapted from Mitchell et al. (2019) to instrumentation and environment:

- **Instrumentation (sensors and bands):**
  - Sentinel-1 GRD (SAR): VV + VH polarizations, 2 channels, 10 m GSD, 264×264 px patches (~2.6 km).
  - Sentinel-2 TOA (optical): 13 bands (wavelengths 440–2190 nm; e.g., B4 665 nm/30 nm, B8 842 nm/115 nm), 10 m GSD.
  - Sentinel-3 OLCI: 21 bands (400–1020 nm; 7.5–40 nm bandwidths), 300 m GSD.
  - Sentinel-5P: single-column variables (tropospheric NO2 column number density, CO, SO2, O3 column number density), ~1 km, 28×28 px.
  - Copernicus DEM GLO-30: elevation, 30 m GSD.
  - Patch sizes differ by modality (16/16/8/4/64 px) following each sensor's resolution.
- **Environment (application domains):**
  - Geographic region: Copernicus-Pretrain covers land + 0.5° coastal buffer, sampled around top-10K populated cities and polar regions; benchmark datasets span Europe (EuroSAT, BigEarthNet subset, LCZ), global land (LC100, DFC2020 test tiles), and global flood events (Kuro Siwo); AQ-S5P covers Europe (EEA 1 km² grid).
  - Cloud cover: S3 pretraining tiles filtered at >20% bright pixels; LC100/Biomass benchmark imagery cloud-filtered with a Cloud-S3-trained model; Cloud-S2/S3 tasks explicitly target cloudy scenes.
  - Season: S1/S2 pretraining patches are 4 seasonal acquisitions; benchmark tasks support static (1 image), pair (change detection) or seasonal (4 image) modes; S5P pretraining uses 12 monthly means.
  - Land cover type: LULC datasets cover 10–23 classes of CORINE-style land cover (EuroSAT, BigEarthNet, LC100, DFC2020) and 17 local climate zones (LCZ).
  - Metadata availability varies by benchmark: DFC2020-S1/S2 and LCZ-S2 lack geolocation and time metadata; EuroSAT-S1/S2 lack time metadata.

#### Metrics

- **mIoU (mean Intersection over Union):** segmentation tasks (Cloud-S2, Cloud-S3, DFC2020-S1/S2, LC100Seg-S3, Flood-S1).
- **OA (Overall Accuracy):** single-label classification tasks (EuroSAT-S1/S2, LCZ-S2).
- **mAP (mean Average Precision):** multi-label classification tasks (BigEarthNet-S1/S2, LC100Cls-S3).
- **RMSE (Root Mean Squared Error):** regression tasks (Biomass-S3 in t/ha AGB; AQ-NO2-S5P and AQ-O3-S5P air quality levels).
- **Protocol:** three-run averages with standard deviations; frozen-encoder evaluation (linear probing with SGD, batch 64; UPerNet decoder with AdamW, batch 16; 50 epochs) against supervised ViT-S/16 and ViT-B/16 baselines trained from scratch for 80 epochs.

### Results

**Benchmark results (Copernicus-Bench, Copernicus-FM ViT-B/16 with modality-adaptive patch sizes; three-run mean ± std):**

| Task | Sensor | Metric | Supervised ViT-B/16 | DOFA | Copernicus-FM |
|---|---|---|---|---|---|
| Cloud-S2 | S2 TOA | mIoU | 59.4 ± 1.0 | 65.0 ± 0.2 | 66.7 ± 0.1 |
| Cloud-S3 | S3 OLCI | mIoU | 63.0 ± 0.8 | 58.2 ± 0.1 | **62.0 ± 0.7** |
| EuroSAT-S1 | S1 GRD | OA | 81.5 ± 0.9 | 81.7 ± 0.1 | **87.2 ± 0.1** |
| EuroSAT-S2 | S2 TOA | OA | 97.6 ± 0.1 | 97.2 ± 0.1 | **97.9 ± 0.1** |
| BigEarthNet-S1 | S1 GRD | mAP | 70.6 ± 0.4 | 70.5 ± 0.0 | 77.9 ± 0.0 |
| BigEarthNet-S2 | S2 SR | mAP | 80.1 ± 0.1 | 75.5 ± 0.0 | 79.0 ± 0.0 |
| LC100Cls-S3 | S3 OLCI | mAP | 91.4 ± 0.5 | 89.5 ± 0.0 | **93.3 ± 0.4** |
| DFC2020-S1 | S1 GRD | mIoU | 50.8 ± 0.5 | 49.7 ± 0.1 | 52.4 ± 0.1 |
| DFC2020-S2 | S2 TOA | mIoU | 66.2 ± 0.7 | 61.8 ± 0.1 | 64.5 ± 0.1 |
| LC100Seg-S3 | S3 OLCI | mIoU | 19.3 ± 0.5 | 16.5 ± 0.1 | **24.1 ± 0.0** |
| Flood-S1 | S1 GRD | mIoU | 78.3 ± 0.3 | 76.0 ± 0.1 | 77.7 ± 0.0 |
| LCZ-S2 | S2 TOA | OA | 85.3 ± 0.8 | 83.0 ± 0.3 | 84.4 ± 0.0 |
| Biomass-S3 | S3 OLCI | RMSE ↓ | 68.3 ± 0.4 | 74.1 ± 0.1 | **66.3 ± 0.1** |
| AQ-NO2-S5P | S5P NO2 | RMSE ↓ | 3.4 ± 0.0 | 3.3 ± 0.0 | **2.8 ± 0.0** |
| AQ-O3-S5P | S5P O3 | RMSE ↓ | 1766.8 ± 22.1 | 1755.6 ± 19.8 | **789.4 ± 2.6** |

Copernicus-FM outperforms supervised training on the majority (11/15) of tasks, despite utilizing fewer trainable parameters and iteration steps, and achieves the best scores on the S3/S5P tasks (no competing foundation model supports those modalities). On BigEarthNet-S1/S2 and LCZ-S2 the specialized baselines SoftCon and the supervised ViT-S/16 remain better.

**Ablation study (ViT-Small, 10K-grid subset, 100 epochs; cumulative component additions):**

| Component | EuroSAT-S1 (OA ↑) | EuroSAT-S2 (OA ↑) | EuroSAT-RGB (mAP ↑) | LC100Cls-S3 (mAP ↑) | AQ-O3-S5P (RMSE ↓) |
|---|---|---|---|---|---|
| Baseline (DOFA) + dynamic patch size | 56.3 | 87.6 | 62.2 | 86.7 | 2218.0 |
| + bandwidth (Fourier encoding) | 56.5 | 88.9 | 65.4 | 87.1 | 1710.7 |
| + variable hypernetwork | 57.5 | 88.9 | 65.8 | 86.6 | 1598.1 |
| + metadata encoding | 77.9 | 88.9 | 78.5 | 90.7 | 839.3 |
| + continual distillation | 81.0 | 89.5 | 78.9 | 90.7 | 811.6 |

Metadata encoding provides the largest single improvement, especially on non-optical modalities (AQ-O3-S5P RMSE −758.8; EuroSAT-S1 OA +22.4).

**Climate grid-embedding regression (linear regression on 10K grids, ERA5 targets; three-run average RMSE):**

| Input | Temperature (°C) mean / std | Precipitation (m) mean / std | Surface press. (Pa) mean / std | Sea-level press. (Pa) mean / std | uu wind (m/s) mean / std | vv wind (m/s) mean / std |
|---|---|---|---|---|---|---|
| Coordinates (raw) | 8.36 ± 0.00 / 3.06 ± 0.00 | 0.79 ± 0.00 / 0.50 ± 0.00 | 7305.91 ± 0.37 / 141.38 ± 0.01 | 290.68 ± 0.02 / 164.84 ± 0.01 | 1.12 ± 0.01 / 0.55 ± 0.00 | 0.95 ± 0.00 / 0.58 ± 0.00 |
| Coordinates (Fourier-encoded) | 3.99 ± 0.00 / 2.09 ± 0.00 | 0.61 ± 0.00 / 0.42 ± 0.00 | 6537.54 ± 1.78 / 101.09 ± 0.18 | 200.93 ± 0.09 / 108.12 ± 0.01 | 0.96 ± 0.01 / 0.51 ± 0.00 | 0.80 ± 0.00 / 0.54 ± 0.00 |
| Copernicus-FM grid embeddings | 2.66 ± 0.01 / 1.72 ± 0.00 | 0.47 ± 0.00 / 0.33 ± 0.00 | 1627.21 ± 30.13 / 97.53 ± 0.71 | 197.17 ± 0.62 / 117.57 ± 0.86 | 0.94 ± 0.00 / 0.43 ± 0.00 | 0.78 ± 0.00 / 0.43 ± 0.00 |
| **Grid embeddings + coordinates** | **1.98 ± 0.01 / 1.35 ± 0.00** | **0.46 ± 0.00 / 0.32 ± 0.00** | **1676.39 ± 33.64 / 78.53 ± 0.08** | **174.80 ± 0.21 / 90.37 ± 0.15** | **0.91 ± 0.00 / 0.42 ± 0.00** | **0.76 ± 0.00 / 0.43 ± 0.00** |
| Coordinates (SatCLIP location encoder) | 3.67 ± 0.03 / 1.58 ± 0.00 | 0.48 ± 0.00 / 0.32 ± 0.00 | 3178.91 ± 8.75 / 97.74 ± 0.19 | 202.76 ± 0.27 / 110.71 ± 0.41 | 0.92 ± 0.00 / 0.43 ± 0.00 | 0.79 ± 0.00 / 0.45 ± 0.00 |

(Grid embeddings are obtained by averaging the model-encoded images of each grid cell; targets are the 10-year mean and standard deviation of each parameter from ERA5.) Grid embeddings consistently outperform coordinate-based baselines, and combining embeddings with coordinates yields the best overall performance on every parameter — e.g., temperature mean RMSE drops from 8.36 (raw coordinates) to 1.98 (embeddings + coordinates).

#### Summary

Copernicus-FM is comparable to or better than state-of-the-art single- or multimodal foundation models on their applicable tasks, and largely improves downstream performance on Sentinel-3 and Sentinel-5P tasks, verifying the benefits of cross-modal pretraining for both surface and atmospheric applications.

## Model Examination [optional]

The paper provides interpretability-oriented analyses: (1) ablations isolating each architectural component (dynamic patch size, bandwidth Fourier encoding, variable hypernetwork, metadata encoding, continual distillation — see Results); (2) metadata ablations showing that location metadata matters most among the three metadata types, and that a higher metadata dropping ratio is better; (3) visualization of the learned Fourier encodings for wavelengths/bandwidths, variable names, and metadata; and (4) global visualization of the Copernicus-Embed-025deg embedding map (PCA to 3-d), showing that raw coordinates produce over-smooth climate predictions while EO-derived grid embeddings capture finer details and extremes.

## Environmental Impact

Carbon emissions can be estimated using the [Machine Learning Impact calculator](https://mlco2.github.io/impact#compute) presented in [Lacoste et al. (2019)](https://arxiv.org/abs/1910.09700).

- **Hardware Type:** NVIDIA A100 GPUs (40 GB).
- **Hours used:** 512 GPU hours per pretraining run (128 node hours on one compute node with 4× A100 40 GB; ~60 hours on 8× A100 according to the official README).
- **Cloud Provider:** Not reported by the authors.
- **Compute Region:** Not reported by the authors.
- **Carbon Emitted:** Not reported by the authors.

## Technical Specifications [optional]

### Model Architecture and Objective

**Architecture.** Unified ViT encoder with sensor-aware dynamic patch embedding:
- **Spectral hypernetwork:** per-channel central wavelength and bandwidth are Fourier-encoded (128-d), passed through a residual FC layer and a 1-layer TransformerEncoder weight generator that emits the 16×16×embed_dim convolution kernel weights and biases for the patch embedding; the kernel is dynamically resized (FlexiViT-style) to each modality's patch size.
- **Variable hypernetwork:** non-spectral variables (S5P NO2/CO/SO2/O3, DEM) are represented by 128-d language embeddings of their variable names (frozen Llama 3.2, computed once offline) and feed the same weight-generator structure.
- **Metadata integration:** geolocation (lon/lat), patch area (km²), and time (days since 1970-01-01) are Fourier-encoded, linearly projected, expanded across patches, and added to the positional embeddings; learnable tokens replace missing metadata (drop probability 0.7).
- **Backbone:** standard timm Transformer blocks (qkv_bias=True, GELU, LayerNorm), MLP ratio 4.0; released variants ViT-B/16 (768-d, 12 blocks, 12 heads) and ViT-L/16 (1024-d, 24 blocks, 16 heads).
- **Pretraining model:** MAE-style encoder-decoder with light decoder (512-d, 8 blocks, 16 heads), dynamic decoder patch predictors, and a distillation projection head.

**Parameter counts (computed from the official code, see below):**

| Component | ViT-B/16 | ViT-L/16 |
|---|---|---|
| Transformer blocks | 85,054,464 | 302,309,376 |
| Spectral hypernetwork (patch embed) | 26,104,064 | 34,591,232 |
| Variable hypernetwork (patch embed) | 26,104,064 | 34,591,232 |
| Metadata MLP encoders (coord/scale/time) | 1,771,776 | 3,148,800 |
| Class token + metadata tokens + final norm | 4,608 | 6,144 |
| **Encoder backbone (trainable)** | **139,038,976** | **374,646,784** |
| Fixed positional embeddings (non-trainable) | 151,296 | 201,728 |

Method: parameters were computed analytically from the architecture definitions in the official code (`CopernicusFMViT` in `src/model_vit.py`, `MaskedAutoencoderViT` in `src/models_mae_cfm.py`, hypernetworks in `src/dynamic_hypernetwork.py`), summing every submodule (each Transformer block contributes 12d²+13d parameters; each hypernetwork consists of a 1-layer TransformerEncoder weight generator (d_model=128) plus FCResLayer and output heads; plus metadata encoders and tokens). The computation was cross-validated against the released checkpoint file sizes: 139,038,976 × 4 bytes ≈ 556 MB, matching the 558 MB ViT-B checkpoint; 374,646,784 × 4 bytes ≈ 1,499 MB, matching the 1.5 GB ViT-L checkpoint — confirming the released weights are the fp32 encoder backbones. The checkpoint files also confirm two released variants: `CopernicusFM_ViT_base_varlang_e100.pth` and `CopernicusFM_ViT_large_varlang_e100.pth` (both at epoch 100).

**Objective.** Pretraining loss = masked image modeling (MAE-style patch reconstruction on flattened tokens, masking ratio 0.7, per-modality dynamic patch predictor) + continual distillation (cosine-similarity loss between student projection and frozen teacher features: DINOv2 ViT-B/14 on S2-derived RGB with weight 0.1; SoftCon ViT-B/14 on S1/S2 full channels with weight 0.2).

### Compute Infrastructure

#### Hardware

- Pretraining: NVIDIA A100 GPUs (40 GB); one run = 512 GPU hours (4× A100 node, 128 node hours; ~60 h on 8× A100 per the official README).
- Benchmark experiments: single GPU, three runs per experiment.

#### Software

- Python 3.10 (recommended conda environment); PyTorch + timm + einops + rasterio + pyproj (+ torchvision, kornia, webdataset, tensorboard, wandb). See `requirements.txt` in the official repository.
- Variable-name embeddings: Llama 3.2 (frozen), pre-encoded offline.

## Citation [optional]

**BibTeX:**

```bibtex
@misc{wang2025unifiedcopernicusfoundationmodel,
      title={Towards a Unified Copernicus Foundation Model for Earth Vision},
      author={Yi Wang and Zhitong Xiong and Chenying Liu and Adam J. Stewart and Thomas Dujardin and Nikolaos Ioannis Bountos and Angelos Zavras and Franziska Gerken and Ioannis Papoutsis and Laura Leal-Taixé and Xiao Xiang Zhu},
      year={2025},
      eprint={2503.11849},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2503.11849},
}
```

**APA:**

Wang, Y., Xiong, Z., Liu, C., Stewart, A. J., Dujardin, T., Bountos, N. I., Zavras, A., Gerken, F., Papoutsis, I., Leal-Taixé, L., & Zhu, X. X. (2025). Towards a Unified Copernicus Foundation Model for Earth Vision. *arXiv preprint arXiv:2503.11849*. Presented at ICCV 2025 (oral).

## Glossary [optional]

- **MIM (Masked Image Modeling):** self-supervised objective that masks random patch tokens and reconstructs them with a light decoder.
- **MAE (Masked Autoencoder):** the MIM architecture (He et al., 2022) on which Copernicus-FM's pretraining is based.
- **Hypernetwork:** a network that generates the weights of another network; here it generates patch-embedding convolution kernels conditioned on the sensor's spectral response (spectral hypernetwork) or variable name (variable hypernetwork).
- **GSD (Ground Sample Distance):** pixel size on the ground (e.g., 10 m for Sentinel-2).
- **Sentinel-1/2/3/5P:** ESA Copernicus missions — S1: C-band SAR; S2: multispectral optical (13 bands, 10 m); S3: OLCI ocean/land color instrument (21 bands, 300 m); S5P: TROPOMI atmospheric chemistry sensor (~1 km columns of NO2, CO, SO2, O3).
- **mIoU:** mean Intersection over Union (segmentation metric). **OA:** Overall Accuracy. **mAP:** mean Average Precision (multi-label classification). **RMSE:** Root Mean Squared Error (regression).
- **LCZ (Local Climate Zone):** climate-based urban/rural zone classification scheme (17 classes).
- **SOMO35:** daily maximum 8-hour mean ozone concentration metric used for the AQ-O3-S5P targets (93.2nd percentile, EEA 2021).
- **ERA5:** ECMWF global reanalysis dataset; the grid structure of Copernicus-Pretrain is aligned to ERA5 coordinates.
- **UPerNet:** segmentation decoder used on top of frozen encoders in Copernicus-Bench.

## More Information [optional]

- Datasets: Copernicus-Pretrain (20.7 TB, `wangyi111/Copernicus-Pretrain`), Copernicus-Bench (`wangyi111/Copernicus-Bench`), and Copernicus-Embed-025deg (`wangyi111/Copernicus-Embed-025deg`; global embedding map 721×1440×768 at 0.25°) are available on Hugging Face under CC-BY-4.0.
- The authors state their intention to contribute the dataset, model, and benchmark to open-source libraries such as TorchGeo.
- Future work stated by the authors: efficient scaling to satellites beyond the Sentinel series, and native model support for multimodal fusion and time series processing; potential incorporation of grid embeddings into ML-based medium-range weather forecasting.

## Model Card Authors [optional]

Maicol Julian Torres Ochoa — mjtorreso@udistrital.edu.co — GitHub: @maicooljt8a-ops (https://github.com/maicooljt8a-ops) — Inteligencia Artificial Aplicada al Territorio, Universidad Distrital Francisco Jose de Caldas.

## Model Card Contact

For questions about this model card: Maicol Julian Torres Ochoa (mjtorreso@udistrital.edu.co). For questions about the model itself, please contact the authors via the official repository (https://github.com/zhu-xlab/Copernicus-FM) or the Hugging Face model page (https://huggingface.co/wangyi111/Copernicus-FM).

## Sources

- Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., & Gebru, T. (2019). *Model Cards for Model Reporting*. FAT* 2019. https://arxiv.org/abs/1810.03993
- Wang, Y., et al. (2025). *Towards a Unified Copernicus Foundation Model for Earth Vision*. ICCV 2025 (oral), arXiv:2503.11849. https://arxiv.org/abs/2503.11849 (latest version: v3, 31 Jul 2025 — https://arxiv.org/html/2503.11849v3)
- Official code repository: https://github.com/zhu-xlab/Copernicus-FM (README, LICENSE, `Copernicus-FM/demo.ipynb`, `Copernicus-FM/src/`)
- Model weights: https://huggingface.co/wangyi111/Copernicus-FM (`CopernicusFM_ViT_base_varlang_e100.pth`, `CopernicusFM_ViT_large_varlang_e100.pth`, `varname_embed/varname_embed_llama3.2_1B.pt`)
- Pretraining dataset card: https://huggingface.co/datasets/wangyi111/Copernicus-Pretrain
- Copernicus-Bench datasets: https://huggingface.co/datasets/wangyi111/Copernicus-Bench
- Copernicus-Embed-025deg dataset: https://huggingface.co/datasets/wangyi111/Copernicus-Embed-025deg
- Hugging Face model card template (structure): https://github.com/huggingface/huggingface_hub/tree/main/src/huggingface_hub/templates
