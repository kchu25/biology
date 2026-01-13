@def title = "Translation Efficiency (TE) Dataset Processing"
@def published = "12 January 2026"
@def tags = ["rna", "screening", "data-processing"]

# Translation Efficiency (TE) Dataset Processing

## Overview

This document describes the process of creating a sequence-to-label dataset for predicting translation efficiency from ribosome profiling data.

## Data Source

- **Ribo-seq data**: `GSM1331342_A1_control_rpf_rep_1.csv` (ribosome profiling)
- **RNA-seq data**: `GSM1331343_A3_control_rna_rep_1.csv` (mRNA sequencing)
- Both files contain 47,087 rows with read counts mapped to different transcript regions

## Processing Workflow

### 1. Data Loading

Loaded paired Ribo-seq and RNA-seq data files containing read counts for:
- 5' leader (5' UTR)
- CDS (Coding Sequence)
- 3' UTR

### 2. Translation Efficiency Calculation

**Translation Efficiency (TE)** = Ribo-seq CDS reads / RNA-seq CDS reads

This ratio indicates how efficiently each transcript is translated into protein.

#### Filtering Steps:
1. **Coding transcripts only**: Filtered for `annotation type == 'coding'`
   - Result: 35,851 coding transcripts in both datasets

2. **Merge datasets**: Matched transcripts by RefSeq accession identifier
   - Result: 35,851 merged transcripts

3. **Minimum read threshold**: Required ≥10 reads in both Ribo-seq and RNA-seq CDS
   - Result: 16,535 transcripts

4. **Outlier removal**: Filtered TE values to reasonable range (0.01 < TE < 100)
   - Result: 16,534 transcripts

#### TE Statistics:
- Min: 0.025
- Max: 99.600
- Mean: 0.688
- Median: 0.571

### 3. Sequence Fetching from NCBI

Fetched sequences and 5' UTR annotations from NCBI using the Entrez API.

#### Key Challenge: No Explicit 5' UTR Annotations

RefSeq records in this dataset **do not have explicit `5'UTR` feature annotations**. They only contain:
- `source` (full sequence range)
- `gene` 
- `exon` features
- `CDS` (coding sequence) with start/end positions
- Misc features (regulatory elements, polyA sites, etc.)

**Solution**: Infer 5' UTR from CDS start position
- The 5' UTR is the sequence **before** the CDS starts
- If CDS starts at position N, then 5' UTR = sequence[0:N]
- Example: For a transcript where CDS starts at position 30, the 5' UTR is nucleotides 0-29 (30 nt total)

#### Sequence Extraction Logic:
```python
# Find the CDS start position
cds_start = None
for feature in record.features:
    if feature.type == "CDS":
        cds_start = int(feature.location.start)
        break

# Infer 5' UTR as everything before CDS
if cds_start is not None and cds_start > 0:
    utr5_seq = str(record.seq[:cds_start])
```

**Note**: This approach works because mRNA transcripts follow the structure: 5' UTR → CDS → 3' UTR

#### API Considerations:
- NCBI rate limit: ~3 requests/second without API key
- Added 0.34 second delay between requests
- Implemented retry logic for failed requests
- Expected time: 10-20 minutes for full dataset

### 4. Final Dataset Structure

Each sample in the dataset contains:
- **gene**: Gene symbol
- **refseq_id**: RefSeq accession identifier (e.g., NM_000356)
- **rpf_cds**: Ribo-seq CDS read count
- **rna_cds**: RNA-seq CDS read count
- **TE**: Translation efficiency (rpf_cds / rna_cds)
- **log2_TE**: Log₂-transformed TE (useful for modeling)
- **utr5_seq**: 5' UTR nucleotide sequence
- **utr5_length**: Length of 5' UTR in nucleotides
- **description**: Transcript description

### 5. Output Files

- **`te_dataset.csv`**: Final dataset with sequences and TE values
- **`Untitled-1.ipynb`**: Interactive notebook with full processing pipeline

## Biological Significance

### Why 5' UTR?

The 5' UTR contains regulatory elements that control translation:
- **Kozak sequence**: Around the start codon, affects translation initiation
- **Secondary structures**: Can block ribosome access
- **uORFs**: Upstream open reading frames that regulate translation
- **IRES elements**: Internal ribosome entry sites

### Why Use CDS for TE?

The CDS (Coding Sequence) is where active translation occurs:
- Ribosomes bind to mRNA and translate the CDS into protein
- Ribo-seq measures ribosome occupancy (translation activity)
- RNA-seq measures mRNA abundance
- Their ratio in the CDS region best reflects translation efficiency

## Gene Structure Reference

```
[5' UTR] --- [CDS: start codon → protein-coding → stop codon] --- [3' UTR]
   ↑              ↑                                                    ↑
Feature    Where translation              Regulatory region
(input)      happens (for TE)             (not translated)
```

## Next Steps

The dataset can be used for:
1. **Regression models**: Predict continuous TE values from 5' UTR sequences
2. **Classification**: Predict high/low translation efficiency categories
3. **Feature analysis**: Identify sequence motifs associated with TE
4. **Deep learning**: Train CNNs or Transformers on sequence data

### Encoding Options:
- One-hot encoding: A=[1,0,0,0], C=[0,1,0,0], G=[0,0,1,0], T=[0,0,0,1]
- k-mer embeddings
- Learned embeddings (similar to NLP word embeddings)

### Normalization:
Consider using log₂(TE) instead of raw TE values to handle the wide dynamic range.

## References

- Processing guide: https://kchu25.github.io/biology/blog/screenings/processing_te/
- Data source: GEO (Gene Expression Omnibus)
- Tool: Biopython for NCBI Entrez API access
