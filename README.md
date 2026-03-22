# ICU Patient Summary Generation with LLMs

**AIM2 Course Project — Group 1**

Generating structured ICU patient summaries from fragmented clinical notes and EHR data using retrieval-augmented generation (RAG) with large language models.


## Quick links

- [Project guide](docs/project_guide_v4_final.md) — full plan, datasets, repos, references
- [Pipeline overview](docs/figures/pipeline_overview_v4.png)
- [Timeline](docs/figures/project_timeline.png)
- [Resource map](docs/figures/resource_map.png)

## Project structure

```
├── docs/               # Project guide, figures, proposal
├── notebooks/          # Exploration and prototyping
├── src/                # Source code (added as we build)
├── configs/            # Experiment configs
├── scripts/            # Runnable scripts
├── data/               # .gitignored — local MIMIC data
└── results/            # .gitignored — experiment outputs
```

## Datasets (require PhysioNet credentials)

- **Primary**: [VeriFact-BHC](https://physionet.org/content/mimic-iii-ext-verifact-bhc/1.0.0/) — 100 MIMIC-III patients
- **Structured data**: [MIMIC-III v1.4](https://physionet.org/content/mimiciii/1.4/)
- **Ablation**: [MIMIC-IV-Ext-BHC](https://physionet.org/content/labelled-notes-hospital-course/1.2.0/)

> ⚠️ No patient data is committed to this repository.

## Setup

```bash
# Create environment
mamba env create -f environment.yml
conda activate 702-project

# Verify
python -c "import torch; print(f'PyTorch {torch.__version__}, MPS: {torch.backends.mps.is_available()}')"
python -c "import transformers; print(f'Transformers {transformers.__version__}')"
python -c "import faiss; print('FAISS ready')"
```
## Drive

Link: https://drive.google.com/drive/folders/1GxRiP60ByW-IuDxoXJYQuZipRSnrh9-R
