# PTR Proteogenomics — Breast Cancer (CPTAC-BRCA)

Post-transcriptional regulation analysis on breast cancer, based on matched transcriptomics, proteomics and copy-number data from the CPTAC-BRCA cohort.

## Research Questions

1. **Correlation analysis** : For each gene, how tightly does protein abundance track its mRNA across patients? Weakly correlated genes are candidates for post-transcriptional
regulation.
2. **Machine learning** : Does genomic dosage (copy-number, CNV) improve protein prediction beyond mRNA alone?

## Dataset

- **Name**: CPTAC-BRCA (Clinical Proteomic Tumor Analysis Consortium — Breast Cancer)
- **Source**: [`cptac`](https://pypi.org/project/cptac/) Python package (public [CPTAC](https://proteomics.cancer.gov/programs/cptac) data)
- **Size**: ~119–121 tumor patients after aligning the omics; ~9,800 genes after filtering
poorly measured proteins. Modalities: transcriptomics (mRNA), proteomics, copy-number (CNV).

The data is not included in this repository (see `.gitignore`). It is downloaded
automatically by the `cptac` package the first time the notebook runs.

## Project Structure
├── data/                              # downloaded automatically by cptac (not versioned)
├── src/
│   └── ptr-proteogenomics-brca.ipynb  # full analysis: preprocessing → correlation → PTR → ML
├── requirements.txt
├── .gitignore
└── README.md

## Methodology

### 1. Preprocessing

Load and align transcriptomics, proteomics and CNV on common patients and genes;
simplify gene identifiers; filter proteins missing in more than 50% of patients.

### 2. Per-gene RNA–protein correlation

Spearman correlation between mRNA and protein across patients (rank-based, robust to
outliers and to the different scales of the two omics). Low correlation flags genes
whose protein is decoupled from mRNA — post-transcriptional regulation candidates.

### 3. Per-patient protein-to-RNA deviation (PTR)

After z-scoring each gene, `PTR = protein_z − mRNA_z` highlights individual tumors that
depart from a gene's usual mRNA–protein relationship, even for well-coupled genes.

### 4. ML — does CNV add information?

A Random Forest predicts protein from mRNA vs mRNA + CNV, scored by 5-fold
cross-validation on held-out patients (patient-level splits, no data leakage).

## Key Results

| Analysis                          | Result                                             |
| --------------------------------- | -------------------------------------------------- |
| Mean RNA–protein correlation      | **≈ 0.45** (consistent with CPTAC literature)      |
| Genes improved by adding CNV      | **93%** of evaluated genes                         |
| Mean R² gain from CNV             | **≈ +0.19**                                        |

Both predictive models keep negative absolute R² (~119 patients, per-gene models), so the
meaningful ML result is the **relative** gain: copy-number carries protein-level signal
beyond mRNA — which correlation alone cannot capture.

**Documented limitation**: some low-correlation genes are secreted proteins of
extra-tumoral (hepatic/plasma) origin, not genuine post-transcriptional regulation — a
known confounder in tumor proteogenomics.

## Installation

git clone <repo-url>
cd ptr-proteogenomics-brca
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

> Requires Python **3.11** (the `cptac` dependency `pyranges` has no prebuilt wheels for
> Python 3.12+ on Windows).

## saUge
jupyter notebook src/ptr-proteogenomics-brca.ipynb

The notebook runs top to bottom and downloads the CPTAC-BRCA data automatically on first run.

## Disclaimer

⚠️ This project is for **educational purposes only**. It is a proof-of-concept on a public
research cohort and is **not a clinical tool**.

## Author

**Juliette Bouli-Mengue** - Clinical Research Associate → Healthcare Data Science
