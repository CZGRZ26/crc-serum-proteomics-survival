# crc-serum-proteomics-survival
End-to-end R/Bioconductor pipeline for LC–MS serum proteomics analysis of colorectal cancer survival (Alive vs Dead) using PRIDE PXD013150.

# Colorectal Cancer Serum Proteomics – Survival Analysis (Alive vs Dead)

This repository contains a complete, reproducible **R/Bioconductor pipeline** for the analysis of **LC–MS–based quantitative serum proteomics data** from **colorectal cancer (CRC) patients**, with a focus on identifying **proteins and biological pathways associated with survival (Alive vs Dead)**.

The analysis is based on the public PRIDE proteomics dataset **PXD013150** and includes:

- Data parsing from Progenesis exports  
- Quality control and normalization  
- Missing-value imputation  
- Differential expression analysis using `limma`  
- GO and KEGG pathway enrichment using `clusterProfiler`  
- Biological interpretation of survival-associated pathways  

This project is designed as a **portfolio-grade biomedical data science project** demonstrating real-world proteomics analysis workflows.

---

## Project Summary

- **Cancer type:** Colorectal cancer (CRC)  
- **Tissue:** Serum  
- **Technology:** LC–MS quantitative proteomics  
- **Total samples analysed:** 89  
  - 58 Alive  
  - 31 Dead  
- **Total quantified proteins:** 224  
- **Primary comparison:** Alive vs Dead survival status  

---

## Data Source

- **PRIDE Accession:** `PXD013150`  
- **Description:** Human serum proteomics from colorectal cancer patients  
- **Raw inputs used in this project are from https://www.ebi.ac.uk/pride/archive/projects/PXD013150:**
  - `Identifications_proteins.csv` – Protein-level Progenesis export  
  - `Comparisions_metadata.xlsx` – Clinical metadata.

 **Raw mass spectrometry files are NOT included in this repository due to size constraints.**  
> Only processed outputs and summary results are version controlled.

---

## Analysis Pipeline Overview

### **1. Data Import & Structuring**
**Script:** `01_import_build_SE_alive_dead.R`

- Parses non-standard Progenesis CSV with multi-row headers  
- Separates protein metadata from intensity values  
- Aligns LC–MS runs with clinical metadata (`MS FILE`)  
- Constructs a `SummarizedExperiment` object:
  - `assays(intensity)` → protein × sample intensity matrix  
  - `colData(group)` → Alive vs Dead  

 Output:
- `proteomics_crc_alive_dead_se_raw.rds`

---

### **2. Normalization, QC & Imputation**
**Script:** `02_normalisation_imputation.R`

- Log2 transformation: `log2(intensity + 1)`  
- Sample-wide median normalization  
- Missing-value filtering (no proteins exceeded 50% missingness)  
- QC visualisations:
  - Total log2 intensity per sample  
  - PCA before and after normalization  
- Missing-value imputation using **MinDet** (`imputeLCMD`)

 Output:
- `proteomics_crc_alive_dead_se_norm_imp.rds`  
- PCA, boxplot QC figures  

---

### **3. Differential Expression Analysis**
**Script:** `03_differential_expression.R`

- Linear modeling with **limma**
- Design: `~ group` (Dead vs Alive)  
- Empirical Bayes moderation  
- Full differential expression table with:
  - log2 fold-change  
  - raw p-values  
  - Benjamini–Hochberg adjusted p-values  

- Visualisations:
  - **Volcano plot**
  - **Heatmap of top 50 differentially expressed proteins**

 Output:
- `DE_alive_vs_dead_proteins.csv`
- `DE_alive_vs_dead_proteins_annotated.csv`
- Volcano + heatmap figures  

> Note: No individual proteins reached FDR < 0.10. This reflects the limited statistical power typical of serum proteomics studies. Therefore, **exploratory pathway enrichment** was performed using the top-ranked proteins.

---

### **4. GO & KEGG Pathway Enrichment**
**Script:** `04_pathway_enrichment.R`

- UniProt → Gene Symbol → EntrezID mapping  
- GO Biological Process enrichment  
- KEGG pathway enrichment  
- Dotplot visualisation of enriched pathways  

 Output:
- `GO_BP_enrichment_alive_vs_dead.csv`
- `KEGG_enrichment_alive_vs_dead.csv`
- Enrichment dotplots  

---

## 🔬 Key Biological Findings

Although no individual proteins passed FDR-adjusted thresholds, **pathway-level analysis revealed strong and highly significant enrichment** in:

### **GO Biological Processes**
-  Complement activation  
-  Blood coagulation  
-  Hemostasis  
-  Wound healing  

### **Top KEGG Pathway**
-  **Complement and coagulation cascades (hsa04610)**

These signals are driven by proteins including:

> **C3, C4A, C5, CFH, CFHR5, CFP, C4BPB, SERPIND1, SERPINF2, F10, F12, APOH, PF4, HRG**

###  Biological Interpretation

These results strongly suggest that **systemic inflammation, innate immunity, and thrombo-inflammatory pathways are associated with poorer survival** in this colorectal cancer serum cohort — fully consistent with known CRC pathophysiology and cancer-associated hypercoagulability.

---
