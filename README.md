# LLM-Based Summarization of Fragmented ICU Clinical Notes

**A Multi-Model, Multi-Evaluator Study of Zero-Shot and Retrieval-Augmented Generation**

BMI 702 / BMIF 203 — *AI in Medicine II (AIM2)* — Group 1
Ming Lin, Alice Wang, Kelvin Mo, Rodrigo Gameiro

This project investigates whether open-source LLMs can generate accurate Brief Hospital Course (BHC) summaries from the multi-source ICU notes that clinicians actually face during handoffs, and whether retrieval-augmented generation (RAG) improves on naive zero-shot prompting once the input budget is matched.

Final report: [docs/other/final_report.pdf](docs/other/final_report.pdf)

## TL;DR

- **Cohort.** VeriFact-BHC: 100 MIMIC-III ICU patients, 4,787 notes across 14 categories, 13,070 clinician-annotated propositions (5,396 Supported).
- **Generation models.** Mistral 7B Instruct v0.3 and Llama 3.1 8B Instruct (4-bit, A100). 2×2 design: {Mistral, Llama} × {Zero-Shot, RAG v3}.
- **RAG pipeline.** BGE-M3 embeddings, FAISS per-patient indices, six clinically motivated queries, budget-matched to the zero-shot context (∼18K vs. ∼31K avg input tokens).
- **Evaluation.** Reference-based metrics (ROUGE, BERTScore), proposition-level fact recall vs. VeriFact ground truth, and an LLM-as-judge (Qwen3-32B) with a five-dimension clinical rubric.
- **Ablation.** MIMIC-IV-Ext-BHC (curated single-document) with Flan-T5-Large to test when retrieval helps.
- **Headline finding.** Budget-matched RAG matches zero-shot on ROUGE-L (0.168 vs. 0.165 Mistral; 0.159 vs. 0.159 Llama) and proposition recall (∼73%) while consuming ~42% less input context. RAG modestly improves judge-rated completeness/overall quality; baseline keeps a small edge in conciseness. On the curated MIMIC-IV-Ext-BHC ablation, RAG underperforms baseline — retrieval is most valuable precisely when source documentation is fragmented enough to require it.

## Repository structure

```
├── docs/
│   └── other/
│       ├── final_report.pdf      # Final term report
│       ├── Midterm.pdf
│       └── Proposal.pdf
├── notebooks/
│   ├── 01_data_exploration.ipynb           # Cohort / token / note-category EDA
│   ├── 02_baseline.ipynb                   # Mistral 7B zero-shot baseline (VeriFact-BHC)
│   ├── 02b_baseline_llama.ipynb            # Llama 3.1 8B zero-shot baseline
│   ├── 03_rag_pipeline.ipynb               # Mistral RAG v1 / v2 / v3 + reference metrics + proposition recall
│   ├── 03b_rag_llama.ipynb                 # Llama RAG v3 (budget-matched)
│   └── copy_of_"04_llm_as_judge_..."  .ipynb  # Qwen3-32B LLM-as-judge (5-dim rubric, all 100 patients)
├── results/
│   └── llm_judge_qwen3_32b_jsonfix_with_rag3_single_only_all_summary_table_flat.csv
├── configs/   scripts/   src/                # Scaffolding (notebooks are the canonical artifacts)
├── data/                                     # .gitignored — local PhysioNet data only
├── environment.yml  requirements.txt  requirements_colab.txt
└── README.md
```

> Notebooks are the canonical implementation; `configs/`, `scripts/`, `src/` are kept for reproducibility scaffolding but most logic lives in the notebooks above.

## Datasets (PhysioNet credentialed access)

- **Primary** — [MIMIC-III-Ext-VeriFact-BHC](https://physionet.org/content/mimic-iii-ext-verifact-bhc/1.0.0/): 100 MIMIC-III ICU patients with clinician-annotated propositions.
- **Source notes** — [MIMIC-III v1.4](https://physionet.org/content/mimiciii/1.4/).
- **Ablation** — [MIMIC-IV-Ext-BHC](https://physionet.org/content/labelled-notes-hospital-course/1.2.0/): curated single-document hospital-course corpus.

> No patient data is committed to this repository. You must obtain the datasets directly from PhysioNet under your own credentialed-access agreement.

## Key results

**Reference-based metrics on VeriFact-BHC (n=100):**

| Metric         | Mistral ZS | Mistral RAG v3 | Llama ZS | Llama RAG v3 |
|----------------|:---------:|:-------------:|:--------:|:------------:|
| ROUGE-L F1     | 0.165     | **0.168**     | 0.159    | 0.159        |
| BERTScore F1   | 0.819     | **0.820**     | 0.813    | **0.814**    |
| Avg input tok. | ∼31K      | ∼18K          | ∼31K     | ∼18K         |
| Truncated pts. | 42 / 100  | —             | 34 / 100 | —            |

**Proposition-level fact recall (Mistral, BGE-M3 cosine ≥ 0.60):** baseline 73.5% vs. RAG v3 72.5% across 5,396 clinician-verified Supported propositions.

**LLM-as-judge (Qwen3-32B) on Mistral conditions:** RAG v3 leads on completeness (2.80) and overall quality (2.80); baseline leads on conciseness (3.39); hallucination scores are compressed across all four conditions (2.95–2.99) — discussed as a calibration / single-judge limitation.

**Ablation — MIMIC-IV-Ext-BHC (Flan-T5-Large, n=100):** RAG underperforms baseline on every metric (ROUGE-L 0.069 vs. 0.092). Retrieval has nothing useful to do when the input is already a curated single document.

See [docs/other/final_report.pdf](docs/other/final_report.pdf) for the full tables, qualitative anchor cases (Patients 72940 and 61157), discussion, and limitations.

## Setup

```bash
# Conda / mamba environment (Apple Silicon-aware)
mamba env create -f environment.yml
conda activate 702-project

# Verify
python -c "import torch; print(f'PyTorch {torch.__version__}, MPS: {torch.backends.mps.is_available()}')"
python -c "import transformers; print(f'Transformers {transformers.__version__}')"
python -c "import faiss; print('FAISS ready')"
```

For Colab (A100 was used for all generation and judge runs), use `requirements_colab.txt`.

## Reproducing the results

The notebooks are written to run end-to-end on a Colab A100 with the PhysioNet datasets mounted from Drive. Suggested order:

1. `01_data_exploration.ipynb` — load VeriFact-BHC, tokenize with the Mistral tokenizer, compute per-note / per-patient token statistics.
2. `02_baseline.ipynb` and `02b_baseline_llama.ipynb` — zero-shot BHC generation with end-truncation when notes exceed the 32K context window.
3. `03_rag_pipeline.ipynb` — chunk (300 tok / 50 overlap), embed with BGE-M3, build per-patient FAISS indices, run RAG v1 → v2 → v3 (budget-matched), score ROUGE/BERTScore and proposition recall.
4. `03b_rag_llama.ipynb` — Llama RAG v3 reusing the Mistral chunk content / FAISS indices; only the generation model and chat template change.
5. `04_llm_as_judge_*.ipynb` — Qwen3-32B (4-bit, deterministic decoding, strict-JSON with repair fallback) scoring all four Mistral conditions on the five-dimension rubric.

## Drive

Working folder (notebooks, intermediate parquets, generated BHCs, judge outputs):
https://drive.google.com/drive/folders/1GxRiP60ByW-IuDxoXJYQuZipRSnrh9-R

## Contributions

Rodrigo Gameiro led the primary VeriFact-BHC pipeline (data, baseline, RAG iterations, cross-model port, proposition recall) and report writing. Alice Wang led the MIMIC-IV-Ext-BHC ablation. Ming Lin and Kelvin Mo built the LLM-as-judge evaluation pipeline and analysis. All authors contributed to study design and interpretation.
