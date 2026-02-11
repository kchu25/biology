@def title = "Experimental Setup: Plant Core Promoter STARR-seq Analysis"
@def published = "11 February 2026"
@def tags = ["screening"]

# Experimental Setup: Plant Core Promoter STARR-seq Analysis

## Overview

This study uses a **massively parallel reporter assay (MPRA)** approach, specifically a **STARR-seq-like** strategy, to quantify the activity of thousands of native plant promoters across three species. The design enables systematic analysis of core promoter elements (CPEs) and transcription factor binding sites (TFBSs) in their native genomic context.

**Publication**: "Synthetic Promoter Designs Enabled by a Comprehensive Analysis of Plant Core Promoters" ([bioRxiv 2021.01.07.425784](https://www.biorxiv.org/content/10.1101/2021.01.07.425784v1))

---

## Experimental Units

### Basic Unit: The Promoter-Barcode Construct

Each experimental unit consists of:
- **Test promoter sequence**: 170bp upstream of TSS (transcription start site)
- **Unique barcode**: Random 12bp DNA sequence embedded in the 3' UTR
- **Reporter gene**: Transcribed mRNA containing the barcode
- **Minimal promoter**: CaMV 35S minimal promoter (control constructs)

### Measurement Level: **Transcription**

The assay measures **transcript abundance** (not translation):
- Barcodes are embedded in the **mRNA transcript** (3' UTR)
- RNA-seq quantifies barcode counts → proxy for transcriptional activity
- Does NOT measure protein levels or translation efficiency

---

## Library Composition

### Three Species-Specific Genomic Promoter Libraries

| Species | Short Code | Promoters in Library | Genome |
|---------|-----------|---------------------|---------|
| *Arabidopsis thaliana* | At | 27,676 | Dicot model |
| *Zea mays* (Maize) | Zm | 38,637 | Monocot crop |
| *Sorghum bicolor* | Sb | 34,173 | Monocot crop |
| **Total** | | **100,486** | |

**Promoter selection criteria**:
- Extracted from genome annotations
- 170bp upstream of annotated TSS
- Unique sequences only (deduplicated)
- Includes protein-coding, miRNA, and other gene types
- Full-length (start position = 1, stop = 170) sequences prioritized in analysis

**Barcode-promoter linking**:
- Multiple barcodes per promoter (technical replicates)
- Median enrichment across barcodes calculated per promoter
- Subassembly process links sequencing reads to specific promoters
- Quality control: only barcodes uniquely mapping to one promoter retained

---

## Assay Systems

### Two Independent Expression Systems

| System | Code | Host Organism | Tissue/Cell Type | Light Conditions |
|--------|------|---------------|------------------|------------------|
| **Tobacco leaf transfection** | leaf | *Nicotiana benthamiana* | Leaf epidermal cells | Dark + Light |
| **Maize protoplast transfection** | proto | *Zea mays* | Mesophyll protoplasts | Dark only |

**Why two systems?**
- Test promoter activity in different cellular contexts
- Leaf: intact tissue, cell-cell interactions preserved
- Protoplast: isolated cells, simpler regulatory environment
- Cross-system validation of promoter strength

---

## Experimental Design

### Conditions Tested

**Factorial design**:
1. **Enhancer**: With/without CaMV 35S enhancer
   - `withENH` (35SEnh): Upstream enhancer present
   - `noENH`: No enhancer (core promoter activity only)
2. **Light**: Dark vs. Light (leaf system only)
   - `dark`: Plants kept in darkness
   - `light`: Plants exposed to light
3. **System**: Leaf vs. Protoplast

**Total experimental conditions per species**:
- Leaf system: 4 conditions (2 enhancer × 2 light)
- Protoplast system: 2 conditions (2 enhancer × 1 dark)
- **6 conditions total per species library**

### Replicates

**Biological replicates**: 2 independent experiments per condition
- Rep1 (Roman numeral I)
- Rep2 (Roman numeral II)

**Technical replicates**: Multiple barcodes per promoter
- Median enrichment across barcodes
- Mean enrichment across biological replicates

---

## Data Files Structure

### Raw Barcode Counts

Located in: `data/barcode_counts/{system}/{species}_Rep{n}/`

**File naming convention**:
```
barcodes_pPSup_{species}PRO_{enhancer}_{rep}_{condition}.count.gz
```

Example:
- `barcodes_pPSup_AtPRO_35SEnh_I_dark.count.gz`
  - Species: Arabidopsis (At)
  - Enhancer: with 35S enhancer (35SEnh)
  - Replicate: 1 (I)
  - Condition: dark

**Three files per replicate**:
1. `inp.count.gz`: Input library (plasmid DNA before transfection)
2. `dark.count.gz`: Output library (mRNA from dark condition)
3. `light.count.gz`: Output library (mRNA from light condition) - leaf only

**File format**:
```
<count> <barcode>
61      AAAAAAAAAAAA
11      AAAAAAAAAAAC
27      AAAAAAAAAAAG
```

### Total Dataset Size

**Native promoter libraries**:
- 3 species × 2 systems × 2 enhancer conditions × 2 replicates = 24 experiments
- Leaf system adds light condition: +12 experiments
- **36 total experiments** (3 input + output files each)

**Additional validation libraries**:
- `PROval`: Validation constructs (synthetic promoters, mutations)
- `PROevo`: Evolved promoters from *in silico* optimization

---

## Enrichment Calculation

### Formula

Promoter strength quantified as **log₂ enrichment**:

```
enrichment = log₂((count_out / Σcount_out) / (count_inp / Σcount_inp))
```

Where:
- `count_out`: Barcode count in output (mRNA)
- `count_inp`: Barcode count in input (plasmid library)
- Sum normalizes for library size

### Normalization Steps

1. **Per-replicate normalization**: Subtract median enrichment of negative control
   - Control: `control-withPRO-noENH` (35S minimal promoter, no enhancer)
   - This sets the minimal promoter baseline to 0
   
2. **Quality filtering**:
   - Read count cutoff: ≥5 reads in both input and output
   - Barcode must uniquely map to one promoter
   - Only full-length (FL) promoters used for modeling
   
3. **Aggregation**:
   - Median enrichment across barcodes (per promoter per replicate)
   - Mean enrichment across biological replicates

**No further normalization applied** before CNN training (raw log₂ values used).

---

## CNN Training Data

### Data Processing Pipeline

**R scripts** (`analysis/`) → **Python files** (`CNN/`)

1. **Raw data** (`data/barcode_counts/`) → `read+process_barcode_counts.R`
2. **Subassembly** links barcodes to promoters → `filter_subassembly.R`
3. **Enrichment calculation** and normalization → `RData/main_data.Rdata`
4. **Motif scoring** adds CPE/TFBS features → `motif_scores.R`
5. **Train/test split** → `modelling.R`
6. **Export to TSV** → `CNN/CNN_train_{system}.tsv`, `CNN_test_{system}.tsv`

### Dataset Files

| File | System | Split | Sequences | Purpose |
|------|--------|-------|-----------|---------|
| `CNN_train_leaf.tsv` | Tobacco leaf | Train | 65,004 | Training leaf model |
| `CNN_test_leaf.tsv` | Tobacco leaf | Test | 7,154 | Evaluating leaf model |
| `CNN_train_proto.tsv` | Maize protoplast | Train | 68,213 | Training protoplast model |
| `CNN_test_proto.tsv` | Maize protoplast | Test | 7,595 | Evaluating protoplast model |
| **Total** | | | **147,966** | |

**Species distribution in training data**:

| Species | Leaf Train | Proto Train | Leaf Test | Proto Test |
|---------|-----------|-------------|-----------|------------|
| At | 15,050 | 15,766 | ~1,670 | ~1,752 |
| Zm | 27,667 | 29,396 | ~3,074 | ~3,266 |
| Sb | 22,287 | 23,051 | ~2,476 | ~2,561 |

### Train/Test Split Strategy

**Selection criteria**:
- Only promoters tested with enhancer in dark condition
- Random 90/10 split per species-system group
- Single random seed (`set.seed(1)`)
- **No stratification** by enrichment value or gene type
- **No separate validation set** (10% validation split used internally during training)

**Data format** (TSV columns):
```
set    sp    gene              sequence                                     enrichment
train  At    AT2G28056         GTTAAGTACCATATAGATGAT...                     1.6878
test   Zm    ENSRNA049996853   TCCTTCCAAACACTTCACCAG...                    -1.8951
```

---

## CNN Model Architecture

### Two Independent Models

Separate models trained for each biological system:
1. **model_leaf**: Predicts activity in tobacco leaf system
2. **model_proto**: Predicts activity in maize protoplast system

**Why separate models?**
- Different cellular contexts may have distinct regulatory logic
- Allows system-specific predictions for *in silico* evolution

### Architecture Details

**Input**: One-hot encoded 170bp sequences → `(170, 4)` tensor

**Layers**:
1. **BiConv1D** (custom bidirectional convolution)
   - 128 filters, kernel size = 13bp
   - 2 stacked layers
   - Forward + reverse strand processing
   - Dropout rate = 0.15
   - **Initialized with CPE/TF motif PWMs** (first 128 filters)

2. **Conv1D**
   - 128 filters, kernel size = 13bp
   - ReLU activation
   - Dropout = 0.15

3. **Flatten** → **Dense(64)** → **BatchNorm** → **ReLU**

4. **Output Dense(1)** (linear activation)

**Training**:
- Loss: Mean Squared Error (MSE)
- Optimizer: Adam (default learning rate)
- Batch size: 128
- Max epochs: 25
- Callbacks:
  - Early stopping (patience = 5)
  - Learning rate reduction (patience = 2)
- Internal validation split: 10% from training data

**No hyperparameter tuning performed** - fixed architecture used.

---

## Key Experimental Specifications

### Promoter Sequences
- **Length**: 170bp upstream of TSS
- **Source**: Genome annotations (RefSeq for Arabidopsis, EGLab for maize/sorghum)
- **Coordinate system**: Position 1 = most upstream, position 170 = TSS

### Barcodes
- **Length**: 12bp random sequences
- **Location**: Embedded in 3' UTR of transcript
- **Purpose**: Unique identifier for each construct copy

### Motifs Used for Model Initialization
- **CPEs** (Core Promoter Elements): TATA-box, Inr, TCT, Y-patch, BRE upstream, BRE downstream
- **TF clusters**: TCP, HSF, NAC binding sites (from clustering analysis)
- **Format**: Position Frequency Matrices (PFMs) converted to PWMs
- **File**: `data/misc/CPEs.meme`, `data/misc/TF-clusters.meme`

### Quality Control Metrics
- **Read count cutoff**: ≥5 reads in input AND output
- **Barcode uniqueness**: Must map to single promoter
- **Promoter completeness**: Full-length (FL) flag for 170bp sequences
- **Replicate concordance**: Mean of 2 biological replicates

---

## Output Metrics

### Primary Measurement
**Promoter strength** = log₂ enrichment relative to 35S minimal promoter

**Interpretation**:
- `enrichment = 0`: Same activity as 35S minimal promoter
- `enrichment > 0`: Stronger than minimal promoter
- `enrichment < 0`: Weaker than minimal promoter
- `enrichment = 1`: 2× stronger
- `enrichment = -1`: 0.5× as strong

### Model Performance Metrics
- **R² (Pearson correlation squared)**: Overall correlation
- **Spearman correlation**: Rank-order correlation
- Calculated overall and per-species
- Reported for both training and test sets

---

## Validation Experiments

Additional constructs tested beyond the genomic libraries:

### Validation Set 1 (Motif Mutations)
- TATA-box mutations (mut/insertion)
- BRE mutations
- Synthetic CPE combinations
- TF binding site insertions

### Validation Set 2 (In Silico Evolution)
- **Starting sequences**: 310 native promoters from all 3 species
- **Optimization targets**: Leaf, protoplast, or both systems
- **Method**: Iterative point mutation + CNN prediction
- **Rounds**: Up to 10 rounds of evolution
- **Constraints**: Remove restriction enzyme sites (BsaI, BbsI)
- **Output**: Optimized promoters for experimental validation

---

## Data Availability

### Processed Data Files
- `RData/main_data.Rdata`: Aggregated enrichment values
- `CNN/*.tsv`: Training/test data for neural networks
- `figures/rawData/*.tsv`: Data tables for publication figures

### Code Organization
- `analysis/`: R scripts for data processing and analysis
- `CNN/`: Python notebooks for model training and evolution
- `barcode_extraction/`: Bash scripts for barcode counting
- `subassembly/`: Scripts to link barcodes to promoters
- `promoter_annotation/`: Extract promoter sequences from genomes

---

## Summary Statistics

| Metric | Value |
|--------|-------|
| **Total promoters tested** | 100,486 |
| **Species** | 3 (Arabidopsis, maize, sorghum) |
| **Expression systems** | 2 (leaf, protoplast) |
| **Experimental conditions** | 6 per species |
| **Biological replicates** | 2 per condition |
| **Total experiments** | 36 |
| **Barcodes per promoter** | Variable (multiple technical replicates) |
| **Training sequences (CNN)** | 133,217 (90% split) |
| **Test sequences (CNN)** | 14,749 (10% split) |
| **Sequence length** | 170bp |
| **Measurement type** | Transcriptional (mRNA abundance) |
| **Normalization baseline** | 35S minimal promoter |

---

## Comparison to Standard MPRA

| Feature | This Study | Typical MPRA |
|---------|-----------|--------------|
| **Library size** | ~100K promoters | 10K-1M variants |
| **Sequence length** | Fixed 170bp | Variable (50-200bp) |
| **Source** | Native genomic promoters | Synthetic/mutated |
| **Barcode location** | 3' UTR (transcribed) | 3' UTR or separate tag |
| **Measurement** | RNA-seq | RNA-seq or DNA-seq |
| **Normalization** | 35S minimal promoter | Various (input DNA) |
| **Systems tested** | 2 (leaf, protoplast) | Usually 1 |
| **Replicates** | 2 biological | 2-3 biological |
| **Train/test split** | 90/10 | Varies (often none) |
| **Model** | CNN (bidirectional) | Various ML approaches |

---

## Critical Design Choices

### Strengths
✅ Large-scale native promoter sampling across 3 species  
✅ Two independent biological systems for validation  
✅ Multiple conditions (enhancer, light) per system  
✅ Biological replicates for statistical robustness  
✅ Motif-initialized CNN leverages prior knowledge  

### Limitations
⚠️ Random train/test split (no stratification)  
⚠️ No separate validation set for hyperparameter tuning  
⚠️ Single random seed (split not robust)  
⚠️ No cross-validation  
⚠️ Fixed CNN architecture (no hyperparameter search)  
⚠️ Species imbalance in datasets (Zm > Sb > At)  
⚠️ Tobacco host system (non-native for all 3 species)  

---

*This document describes the experimental and computational methodology for the plant core promoter analysis. For detailed methods, refer to the original publication and repository code.*
