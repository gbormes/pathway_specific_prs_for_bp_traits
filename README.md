# pathway_specific_prs_for_bp_traits
Reproducible PLINK-based pipeline for constructing and evaluating polygenic risk scores (PRS) for systolic blood pressure using 1000 Genomes–imputed data.

# Polygenic Risk Score (PRS) Construction for Systolic Blood Pressure (SBP)

This repository contains the full analytic pipeline used to construct, tune, and evaluate polygenic risk scores (PRS) for systolic blood pressure (SBP) using genotype data imputed to the 1000 Genomes (1KG) reference panel. The pipeline is identical for diastolic blood pressure, pulse pressure, and mean arterial pressure. 

The workflow follows a standard **base → validation → target cohort** design and uses LD-based clumping and p-value thresholding.

---

## Overview of Analysis Pipeline

1. **GWAS in base (training) cohort**
2. **Reformat PLINK2 GWAS output**
3. **LD-based clumping using validation cohort**
4. **P-value thresholding**
5. **PRS construction in validation cohort**
6. **Selection of optimal p-value threshold**
7. **PRS evaluation in independent target cohort**

---

## Software Requirements

- **PLINK 2.0** (GWAS in base cohort)
- **PLINK 1.9** (LD clumping and PRS scoring)
- **R ≥ 4.0**

Required R packages:
- `stats` (base)
- `utils` (base)

---

## Cohorts and File Naming

| Role | Cohort Prefix | Description |
|----|----|----|
| Base (training) | `pre02a_nex1KG` | Used for GWAS |
| Validation | `pre02a_ngr1KG` | Used for LD clumping and PRS tuning |
| Target | `pre02a_nax1KG` | Independent evaluation cohort |

All cohorts are assumed to be from the same ancestry group.

---

## Input Files

### Genotype Data
PLINK binary files (`.bed`, `.bim`, `.fam`) for each cohort:
pre02a_nex1KG.*
pre02a_ngr1KG.*
pre02a_nax1KG.*


### Phenotype & Covariate Files
Tab-delimited files containing phenotype and covariates:
pre02a_nex1KG.cov
pre02a_ngr1KG.cov
pre02a_nax1KG.cov


Required columns include:
- `FID`, `IID`
- `rsbp` (raw SBP phenotype)
- `SBP15` (adjusted/transformed SBP)
- `age`, `sex10`
- `PC1–PC5`

---

## GWAS Output

GWAS is performed in the base cohort using PLINK 2.0:
pre02a_nex1KG_train_rsbp.rsbp.glm.linear


This file is reformatted to a PLINK 1.9–compatible association file:
pre02a_nex1KG_train_rsbp.assoc.linear


This file is used for all downstream PRS analyses.

---

## LD Clumping

LD clumping is performed using the **validation cohort** as the LD reference:

- Window size: 250 kb
- LD threshold: r² < 0.1
- Index SNP p-value threshold: 1.0

Output files:
pre02a_ngr1KG_clumped_rsbp.clumped
pre02a_ngr1KG_clumped_rsbp.valid.snp


The file `*.valid.snp` contains the final list of **independent SNPs** used for PRS construction.

---

## P-Value Thresholding

PRS are generated using the following p-value thresholds:

0.001
0.0025
0.005
0.0075
0.01
0.025
0.05


Threshold ranges are defined in:
rsbp_range_list


---

## PRS Construction (Validation Cohort)

PRS are constructed in the validation cohort using:

- Clumped SNP list
- GWAS effect sizes from base cohort
- Additive scoring (`--score sum`)

Output:
ngr1KG_rsbp_prs.<threshold>.profile


---

## Selection of Optimal Threshold

For each threshold, the incremental variance explained by the PRS is calculated as:
ΔR² = R²(full model) − R²(null model)


Models include:
- Age
- Sex
- Genetic PCs (PC1–PC5)

The optimal threshold is selected as the one with the maximum ΔR² and saved as:
sbp_1KG_selected_range


Full tuning results are saved as:
ngrsbp_1KG_prs.RDS


---

## PRS Evaluation (Target Cohort)

The selected PRS model is applied to the independent target cohort:
nax1KG_rsbp_prs.profile


PRS performance is evaluated using linear regression with the same covariates as in the validation cohort.

Final results are written to:
sbp_1KG_final_PRS.txt


---

## Output Summary

| File | Description |
|----|----|
| `*.assoc.linear` | GWAS summary statistics |
| `*.clumped` | LD clumping results |
| `*.valid.snp` | Independent SNP list |
| `*.profile` | PRS scores |
| `ngrsbp_1KG_prs.RDS` | PRS tuning results |
| `sbp_1KG_final_PRS.txt` | Target cohort PRS performance |

---

## Notes

- All PRS are constructed using additive genetic effects
- LD clumping is performed using the validation cohort to avoid information leakage.
- This pipeline is easily extensible to pathway-specific PRS by restricting SNP input files.

---

## Citation

If you use or adapt this pipeline, please cite the associated manuscript and reference PLINK appropriately:

- Purcell S, Neale B, Todd-Brown K, Thomas L, Ferreira MAR, Bender D, et al. PLINK: a tool set for whole-genome association and population-based linkage analyses. Am J Hum Genet. 2007 Sep;81(3):559–75.

- Chang CC, Chow CC, Tellier LC, Vattikuti S, Purcell SM, Lee JJ. Second-generation PLINK: rising to the challenge of larger and richer datasets. GigaScience. 2015;4:7. 

---

## Contact

For questions or clarifications, please open an issue or contact the repository maintainer.
