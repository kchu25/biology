@def title = "Cao et al. (2021) 5' UTR Expression Units Explained"
@def published = "19 January 2026"
@def tags = ["data-processing"]

# Cao et al. (2021) 5' UTR Expression Units Explained

## The Datasets You're Asking About

Based on the blog post, "jek", "muscle", and "PCB" likely refer to the **training datasets** they used, which came from:

1. **HEK** (not "jek") = HEK 293T cells
2. **muscle** = Human muscle tissue  
3. **PC** (not "PCB") = PC3 cells (prostate cancer)

These were **NOT their experimental data** - they're published datasets from other labs used to train their computational model.

## Units in the Original Training Datasets

### Translation Efficiency (TE) - The Core Training Metric

**Formula:**
$$
\text{TE} = \frac{\text{Ribo-seq RPKM}}{\text{RNA-seq RPKM}}
$$

**What it measures:**
- **Ribo-seq RPKM** = ribosome footprint counts (how much translation is happening)
- **RNA-seq RPKM** = mRNA transcript counts (how much mRNA exists)
- **TE** = ratio of the two (translation per mRNA molecule)

**Units:** Dimensionless ratio (RPKM / RPKM)

**Interpretation:** Higher TE = more ribosomes per mRNA = more protein output

**Example:** TE = 2.0 means 2× more ribosomes per mRNA compared to TE = 1.0

### Where These Datasets Came From

| Dataset | Source | GEO Accession | What They Measured |
|---------|--------|---------------|-------------------|
| **HEK 293T** | Andreev et al. (2015) | GSE55195 | Translation during stress response |
| **PC3** (prostate cancer) | Hsieh et al. (2012) | GSE35469 | mTOR signaling effects on translation |
| **Muscle** | Wein et al. (2014) | GSE56148 | Translation in muscle tissue |
| **Muscle (GTEx)** | GTEx database | - | mRNA expression levels |

**Key point:** These are publicly available datasets - the authors mined existing high-quality data rather than generating their own training data.

## Units in Their Screening Dataset

### During High-Throughput Screening

**1. FACS (Flow Cytometry) Input:**
- **What:** GFP fluorescence intensity
- **Unit:** Arbitrary fluorescence units (AFU) - photon counts from detector
- **How used:** Cells sorted into bins (top 2.5%, 2.5-5%, 5-10%, 0-100%)

**2. Next-Generation Sequencing Output:**
- **What:** DNA sequencing read counts for each 5' UTR
- **Unit:** Raw read counts (e.g., "this UTR appeared 1,247 times")
- **Normalized:** Counts per million reads

**3. Enrichment Score (Final Metric):**
$$
\text{Enrichment} = \log_2\left(\frac{\text{reads in top bin}}{\text{reads in control bin}}\right)
$$

**Unit:** log₂(dimensionless ratio)

**Where in paper:** Figure 3c shows these log₂ enrichment scores

### During Validation

**1. GFP Expression (Fluorescent Reporters):**
- **Raw measurement:** Mean fluorescence intensity (MFI) from flow cytometry
- **How calculated:** 
  $$
  \text{Relative Expression} = \frac{\text{GFP}_{\text{test UTR}} / \text{BFP}_{\text{test UTR}}}{\text{GFP}_{\text{pVAX1}} / \text{BFP}_{\text{pVAX1}}} \times 100\%
  $$
- **Unit:** Percentage (%) where pVAX1 = 100%
- **Interpretation:** 158% means 1.58× more protein than pVAX1 control

**Where in paper:** Figures 4c-d, Figure 5

**2. Therapeutic Protein Expression (VEGF, CCL21):**
- **Raw measurement:** ELISA
- **Unit:** **pg/mL** (picograms per milliliter)
- **Normalized to:** per 100,000 cells, per 24 hours
- **Reported as:** Percentage relative to pVAX1

**Example from Figure 4f (CCL21):**
- pVAX1 might produce 500 pg/mL
- NeoUTR3 produces 2,760 pg/mL  
- Reported as: 552% (5.52× increase)

**Where in paper:** Figures 4e-f

## The Complete Measurement Pipeline

### Training Phase (Before Experiments)
```
Public datasets (Ribo-seq + RNA-seq)
    ↓
Calculate TE = Ribo-seq RPKM / RNA-seq RPKM
    ↓
Extract 5' UTR features (k-mers, structure, etc.)
    ↓
Train Random Forest model
    ↓
Use genetic algorithm to design synthetic 5' UTRs
```

### Screening Phase (Their Experiments)
```
~12,000 5' UTRs cloned into GFP reporter plasmids
    ↓
Recombinase-mediated integration (1 copy per cell)
    ↓
FACS: Sort by GFP fluorescence → bins
    ↓
DNA-seq: Count reads per UTR in each bin
    ↓
Calculate: log₂(top bin reads / control reads)
    ↓
Rank UTRs by enrichment score
```

### Validation Phase
```
Top candidate UTRs
    ↓
Test individually in cells
    ↓
Measure: GFP/BFP ratio by flow cytometry
OR: Protein concentration by ELISA (pg/mL)
    ↓
Normalize to pVAX1 = 100%
    ↓
Report: "NeoUTR3 gives 158% expression"
```

## Summary Table

| Measurement Type | Raw Unit | Normalized Unit | Where Used |
|-----------------|----------|-----------------|------------|
| **Training data (TE)** | Ribo-seq RPKM / RNA-seq RPKM | Dimensionless ratio | Model training |
| **Screening (FACS)** | Arbitrary fluorescence units | Binned by intensity | Cell sorting |
| **Screening (NGS)** | Read counts | log₂(enrichment ratio) | UTR ranking |
| **Validation (GFP)** | Mean fluorescence intensity | % of pVAX1 (=100%) | Figures 4c-d, 5 |
| **Validation (ELISA)** | pg/mL per 100k cells/24h | % of pVAX1 (=100%) | Figures 4e-f |

## Key Differences from Other Papers

**Yeast (de Boer 2019):**
- Unit: log₂(YFP/RFP) 
- Both on same plasmid

**E. coli (Johns 2018):**
- Unit: log₂(RNA/DNA)
- DNA-seq normalizes for cell count

**This paper (Cao 2021):**
- Training: Translation Efficiency (TE) from public data
- Screening: log₂(reads enrichment) 
- Validation: % of pVAX1 (from GFP/BFP ratio or ELISA pg/mL)

## Where to Find This in the Paper

**Methods Section:**
- "Translation efficiency was calculated as Ribo-seq RPKM / RNA-seq RPKM"
- "ELISA measurements reported as pg/mL per 100,000 cells per 24h"

**Results:**
- Figure 3c: Enrichment scores (log₂ ratios)
- Figure 4: Relative expression (% of pVAX1)
- Figure 5: Multi-cell-type validation

## Bottom Line

**Training datasets (HEK, muscle, PC3):**
- Unit: Translation Efficiency (TE) = dimensionless Ribo-seq/RNA-seq ratio
- Source: Published datasets, not their own data

**Final screening dataset:**
- Screening metric: log₂(enrichment) from NGS read counts
- Validation metric: % of pVAX1 from either GFP/BFP fluorescence ratio OR protein concentration (pg/mL)