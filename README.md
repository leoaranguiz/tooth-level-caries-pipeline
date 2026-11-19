# Two-stage deep learning pipeline for tooth-level caries screening in smartphone intraoral photographs

Code of the article:

> Aranguiz-Gonzalez L, Flores-Cartes R, Tobar-Marambio E, Villena F. A two-stage deep learning pipeline for tooth-level caries screening in smartphone intraoral photographs of children and young people: development and internal validation. *BMC Medical Informatics and Decision Making* (under review).

- **Stage 1:** tooth segmentation with FDI numbering (U-Net++ R50, DeepLabV3+ R50, SegFormer-B2, YOLO11m-seg).
- **Stage 2:** Merged-ICDAS classification of tooth crops (Swin-T, ConvNeXt V2-T, ResNet-50, CoAtNet-0, MaxViT-T, ViT-S/16, YOLO11m-cls).
- Five-fold cross-validation grouped by child in both stages, chained evaluation of the two stages, and a demonstration on 10 reserved children.

## Data

| What | Where |
|---|---|
| Source photographs (Ahmed et al., CC BY 4.0) | https://doi.org/10.5281/zenodo.14827784 |
| Tooth masks, labels, folds and out-of-fold predictions of this study (CC BY 4.0) | Zenodo, DOI to be added |
| Trained weights | Not published; available from the corresponding author on reasonable request |

No photograph, image crop or model weight is stored in this repository, and the notebook outputs were cleared.

## How the notebooks were run

All notebooks ran on Kaggle (Python 3.13; GPU T4 for training and the pipeline, CPU for the analyses). Each one reads its inputs from `/kaggle/input` (the outputs of earlier notebooks, attached as inputs) and writes to `/kaggle/working`. To run them elsewhere, change the input and output folders at the top of each notebook. Main package versions are in `requirements.txt`.

The column "Kaggle notebook" gives the name used in the provenance columns of Additional file 3 of the article. Comments, some variable names and some column names are in Spanish.

The notebooks of section 1 build the curated datasets from the source photographs and from intermediate label files that are not published (they contain annotation-process records). Their final outputs (masks, labels, crop boxes and folds) are in the Zenodo record.

## Order

### 1. Datasets
| # | Notebook | Kaggle notebook | Inputs | What it does |
|---|---|---|---|---|
| 1.1 | `notebooks/11_curated_datasets.ipynb` | `datasets-limpios-icdas-segmentacion` | source photographs; intermediate label files | Child identifiers, folds grouped by child, stage 1 photographs and masks |
| 1.2 | `notebooks/12_reserved_children.ipynb` | `seleccion-reserva-pipeline` | source photographs; 1.1 | Pre-specified selection of the 10 reserved children |
| 1.3 | `notebooks/13_stage2_crops.ipynb` | `base-icdas-v4` | source photographs; earlier crop base | 3466 crops at native resolution, labels and folds |
| 1.4 | `notebooks/14_children_in_both_stages.ipynb` | `ninos-68-vs-81` | 1.1, 1.3 | Children in both datasets and in the chained evaluation |

### 2. Training (GPU)
| # | Notebook | Kaggle notebook | Inputs | What it does |
|---|---|---|---|---|
| 2.1 | `notebooks/21_stage1_segmentation_cv.ipynb` | `benchmark-segmentacion-cv-nino` | 1.1 | Four segmentation models, 5 folds; out-of-fold masks and metrics with 95% CI |
| 2.2 | `notebooks/22_stage2_classification_cv.ipynb` | `benchmark-icdas-v4` | 1.3 | Seven classifiers, 5 folds; out-of-fold predictions and metrics with 95% CI |

### 3. Analyses (CPU, except 3.1 and 3.7)
| # | Notebook | Kaggle notebook | Inputs | What it does |
|---|---|---|---|---|
| 3.1 | `notebooks/31_stage1_map.ipynb` | `map-segmentadores` | 1.1, 2.1 | Mask and box mAP (COCO); cache of instances and pixel confusion matrices |
| 3.2 | `notebooks/32_stage1_identification_iou05.ipynb` | `f1-iou05` | 1.1, 2.1, 3.1 | FDI identification with IoU > 0.5 (main criterion) and sensitivity to the criterion |
| 3.3 | `notebooks/33_stage1_box_iou.ipynb` | `iou-caja-segmentadores` | 1.1, 2.1, 3.1 | Box IoU per tooth |
| 3.4 | `notebooks/34_stage1_dice_raw_yolo.ipynb` | `dice-yolo-crudo` | 1.1, 2.1, 3.1 | Dice of YOLO11m-seg from raw instances |
| 3.5 | `notebooks/35_stage1_dice_no_third_molars.ipynb` | `dice-sin-terceros-molares` | 1.1, 2.1, 3.1 | Dice without third molars |
| 3.6 | `notebooks/36_stage1_error_analysis.ipynb` | `errores-segmentacion-v3` | 1.1, 2.1, 3.1, 3.2 | Wrong number, insufficient contour, not detected; mixed dentition |
| 3.7 | `notebooks/37_stage2_analysis.ipynb` | `analisis-icdas-v4` | 1.3, 2.2 | Classifier selection by validation, confusion matrices, ROC, calibration, Grad-CAM, efficiency |
| 3.8 | `notebooks/38_stage2_rereading_sensitivity.ipynb` | `sensibilidad-relectura-v4` | 1.3, 2.2 | Sensitivity analysis of the model-guided re-reading |

### 4. Pipeline (GPU)
| # | Notebook | Kaggle notebook | Inputs | What it does |
|---|---|---|---|---|
| 4.1 | `notebooks/41_chained_evaluation_and_demo.ipynb` | `pipeline-icdas-v4` | source photographs; 1.1, 1.2, 1.3, 2.1, 2.2 | Chained evaluation (908 teeth, 68 children), odontograms and demonstration on the reserved children |

### 5. Supplementary tables and data package
| # | Notebook | Kaggle notebook | Inputs | What it does |
|---|---|---|---|---|
| 5.1 | `notebooks/51_additional_file_3.ipynb` | `additional-file-3` | outputs of 2.1–4.1 | Builds Additional file 3 (.xlsx) |
| 5.2 | `notebooks/52_zenodo_package.ipynb` | `paquete-zenodo` | outputs of 1.1–4.1 | Builds the files of the Zenodo record and runs the reproduction and privacy checks |

## Reproducing the results without training

Download the Zenodo record. All stage 2 metrics can be recomputed from `stage2_oof_predictions.csv`, the stage 1 identification and error results from `stage1_reference_teeth.csv.gz`, and the chained evaluation from `chained_evaluation.csv`. The README of the record explains each file and how the Dice of Table 2 is aggregated.

## Licence

The code in this repository is released under the MIT licence (see `LICENSE`).

The notebooks that run YOLO11 models (2.1, 2.2, 3.1, 3.7 and 4.1) use the [Ultralytics](https://github.com/ultralytics/ultralytics) package, which is distributed under the AGPL-3.0 licence. Anyone who uses or distributes those parts must comply with the Ultralytics licence; the MIT licence of this repository does not change it. The other libraries used (PyTorch, timm, segmentation-models-pytorch, Transformers, Albumentations, NumPy, pandas, OpenCV, scikit-learn, pycocotools) have permissive licences.

The data are released separately under CC BY 4.0 (see the Zenodo record).

## How to cite

See `CITATION.cff`. Please cite the article and, for the data, the Zenodo record and the source dataset of Ahmed et al.

## Competing interests

L.A.G. is the founder and director of Odontify SpA, and E.T.M. collaborates with Odontify SpA as an advisor (see the article).
