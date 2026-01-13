@def title = "Translation Efficiency Processing: Code Explanation"
@def published = "12 January 2026"
@def tags = ["data-processing"]

# Translation Efficiency Processing: Code Explanation

This document provides detailed explanations of the essential code components used to process ribosome profiling data and create a sequence-to-label dataset.

---

## 1. Import Libraries and Setup

```python
import pandas as pd
import numpy as np
from Bio import Entrez, SeqIO
import time
import matplotlib.pyplot as plt
import seaborn as sns

# Set your email for NCBI (REQUIRED by NCBI API)
Entrez.email = "your.email@example.com"
```

**What this does:**
- `pandas`: Data manipulation and analysis (loading CSV files, merging datasets)
- `numpy`: Numerical operations (log transformations)
- `Bio.Entrez`: Interface to NCBI's Entrez databases (fetches sequences)
- `Bio.SeqIO`: Parse GenBank records
- `time`: Rate limiting for NCBI API
- `matplotlib/seaborn`: Data visualization

**Why Entrez.email is required:** NCBI tracks API usage by email. If you make excessive requests, they'll contact you instead of blocking all users.

---

## 2. Load Data Files

```python
rpf_file = 'GSM1331342_A1_control_rpf_rep_1.csv'
rna_file = 'GSM1331343_A3_control_rna_rep_1.csv'

rpf_data = pd.read_csv(rpf_file, sep='\t')
rna_data = pd.read_csv(rna_file, sep='\t')
```

**What this does:**
- Loads tab-separated CSV files into pandas DataFrames
- `rpf_data`: Ribosome profiling (Ribo-seq) read counts
- `rna_data`: RNA sequencing (RNA-seq) read counts

**Key columns:**
- `Gene`: Gene symbol
- `RefSeq accession identifier`: Unique transcript ID (e.g., NM_000356)
- `annotation type`: Type of transcript (coding, transcript, pseudo)
- `raw mapped reads, CDS`: Number of sequencing reads mapped to the coding sequence

---

## 3. Filter for Coding Transcripts

```python
rpf_coding = rpf_data[rpf_data['annotation type'] == 'coding'].copy()
rna_coding = rna_data[rna_data['annotation type'] == 'coding'].copy()
```

**What this does:**
- Filters out non-coding transcripts (pseudogenes, non-coding RNAs)
- `.copy()` creates independent DataFrames to avoid modifying the original data

**Why coding only?** 
- Translation efficiency is only meaningful for protein-coding genes
- Non-coding RNAs don't have ribosomes actively translating them

---

## 4. Merge Paired Datasets

```python
merged = pd.merge(
    rpf_coding[['Gene', 'RefSeq accession identifier', 'raw mapped reads, CDS']],
    rna_coding[['RefSeq accession identifier', 'raw mapped reads, CDS']],
    on='RefSeq accession identifier',
    suffixes=('_rpf', '_rna')
)
```

**What this does:**
- Joins Ribo-seq and RNA-seq data by matching RefSeq IDs
- `suffixes=('_rpf', '_rna')`: Renames duplicate columns to distinguish sources
  - `raw mapped reads, CDS_rpf`: Ribo-seq CDS reads
  - `raw mapped reads, CDS_rna`: RNA-seq CDS reads

**Result:** Each row contains both Ribo-seq and RNA-seq counts for the same transcript.

---

## 5. Convert to Numeric and Handle Missing Values

```python
merged['rpf_cds'] = pd.to_numeric(merged['raw mapped reads, CDS_rpf'], errors='coerce').fillna(0)
merged['rna_cds'] = pd.to_numeric(merged['raw mapped reads, CDS_rna'], errors='coerce').fillna(0)
```

**What this does:**
- `pd.to_numeric(..., errors='coerce')`: Converts strings to numbers; invalid values become NaN
- `.fillna(0)`: Replaces NaN with 0 (transcripts with no reads)

**Why needed?** 
- Some CSV entries may be empty strings or non-numeric
- Ensures we can perform mathematical operations (division for TE)

---

## 6. Filter by Read Count Threshold

```python
min_reads = 10
filtered = merged[
    (merged['rpf_cds'] >= min_reads) & 
    (merged['rna_cds'] >= min_reads)
].copy()
```

**What this does:**
- Keeps only transcripts with ≥10 reads in **both** Ribo-seq and RNA-seq
- `&` is the element-wise AND operator for pandas

**Why filter?**
- Low read counts produce noisy, unreliable TE estimates
- Example: 1 read / 2 reads = TE of 0.5, but this is statistically meaningless
- Requiring ≥10 reads ensures more stable measurements

---

## 7. Calculate Translation Efficiency

```python
filtered['TE'] = filtered['rpf_cds'] / filtered['rna_cds']
```

**What this does:**
- Calculates TE as the ratio of Ribo-seq to RNA-seq reads
- Element-wise division creates a new column

**Interpretation:**
- TE = 1.0: Translation rate matches mRNA abundance (typical)
- TE > 1.0: More ribosomes than expected (high translation)
- TE < 1.0: Fewer ribosomes than expected (low translation)

---

## 8. Remove Outliers

```python
filtered = filtered[
    (filtered['TE'] > 0.01) & 
    (filtered['TE'] < 100)
].copy()
```

**What this does:**
- Removes extreme TE values that are likely errors
- Keeps TE values between 0.01 and 100

**Why?**
- TE < 0.01: Extremely low, possibly technical artifacts
- TE > 100: Suspiciously high, likely measurement errors or annotation issues
- Biologically reasonable range helps model training

---

## 9. Fetch Sequences from NCBI

### Core Function

```python
def fetch_sequence_with_utr(refseq_id):
    """
    Fetch sequence and 5' UTR from NCBI for a RefSeq ID.
    
    Strategy:
    1. First, look for explicit 5'UTR annotation
    2. If not found, infer from CDS start position (fallback)
    """
    handle = Entrez.efetch(
        db="nucleotide", 
        id=refseq_id, 
        rettype="gb", 
        retmode="text"
    )
    record = SeqIO.read(handle, "genbank")
    handle.close()
    
    utr5_seq = None
    
    # First, try to find explicit 5'UTR feature
    for feature in record.features:
        if feature.type == "5'UTR":
            utr5_seq = str(feature.extract(record.seq))
            break
    
    # If not found, infer from CDS start (fallback)
    if utr5_seq is None:
        for feature in record.features:
            if feature.type == "CDS":
                cds_start = int(feature.location.start)
                if cds_start > 0:
                    utr5_seq = str(record.seq[:cds_start])
                break
    
    return utr5_seq
```

**What this does step-by-step:**

1. **`Entrez.efetch()`**: Downloads GenBank record from NCBI
   - `db="nucleotide"`: Query the nucleotide database
   - `id=refseq_id`: The RefSeq accession (e.g., "NM_000356")
   - `rettype="gb"`: Return GenBank format (includes annotations)
   - `retmode="text"`: Plain text format

2. **`SeqIO.read(handle, "genbank")`**: Parses GenBank record
   - Extracts sequence and all feature annotations
   - `record.seq`: The actual DNA/RNA sequence
   - `record.features`: List of annotated features (genes, CDS, exons, etc.)

3. **Strategy 1 - Look for explicit 5'UTR annotation** (preferred):
   ```python
   for feature in record.features:
       if feature.type == "5'UTR":
           utr5_seq = str(feature.extract(record.seq))
           break
   ```
   - Searches for explicit `5'UTR` feature annotation
   - `feature.extract(record.seq)`: Extracts the exact subsequence
   - `break`: Stops after finding first 5'UTR

4. **Strategy 2 - Infer from CDS start** (fallback):
   ```python
   if utr5_seq is None:
       for feature in record.features:
           if feature.type == "CDS":
               cds_start = int(feature.location.start)
               if cds_start > 0:
                   utr5_seq = str(record.seq[:cds_start])
               break
   ```
   - If no explicit 5'UTR found, look for CDS feature
   - Extract sequence from start to CDS beginning
   - `cds_start > 0`: Ensures there's actually a 5'UTR region

---

## 10. Batch Fetch with Rate Limiting

```python
sequences = []
for i, (idx, row) in enumerate(sample_data.iterrows(), 1):
    refseq_id = row['RefSeq accession identifier']
    
    utr5_seq = fetch_sequence_with_utr(refseq_id)
    
    if utr5_seq:
        sequences.append({
            'gene': row['Gene'],
            'refseq_id': refseq_id,
            'rpf_cds': row['rpf_cds'],
            'rna_cds': row['rna_cds'],
            'TE': row['TE'],
            'log2_TE': np.log2(row['TE']),
            'utr5_seq': utr5_seq,
            'utr5_length': len(utr5_seq)
        })
    
    time.sleep(0.34)  # NCBI rate limit: ~3 requests/second
```

**What this does:**

1. **Loop through filtered transcripts**: `sample_data.iterrows()` iterates row by row

2. **Fetch sequence**: Calls `fetch_sequence_with_utr()` for each RefSeq ID
   - Returns 5' UTR sequence string or None

3. **Validate 5' UTR**: `if utr5_seq:`
   - Only include transcripts where we successfully extracted a 5' UTR
   - Excludes:
     - Transcripts with CDS starting at position 0 (no 5' UTR)
     - Failed fetches (network errors, invalid IDs)

4. **Create dataset entry**: Combines TE data with sequence
   - `gene`: Gene symbol
   - `refseq_id`: RefSeq accession
   - `rpf_cds`, `rna_cds`: Read counts
   - `TE`: Translation efficiency
   - `log2_TE`: Log₂ transformation of TE (useful for modeling, handles wide range)
   - `utr5_seq`: The 5' UTR nucleotide sequence (feature for ML)
   - `utr5_length`: Length of 5' UTR in nucleotides

5. **Rate limiting**: `time.sleep(0.34)`
   - NCBI limits: 3 requests/second without API key
   - 0.34 seconds ≈ 2.94 requests/second (safely under limit)
   - Prevents IP bans from excessive requests

---

## 11. Save the Dataset

```python
sequences_df = pd.DataFrame(sequences)
sequences_df.to_csv('te_dataset.csv', index=False)
```

**What this does:**
- Converts list of dictionaries to pandas DataFrame
- Saves as CSV file for later use
- `index=False`: Don't save row numbers as a column

**Final dataset columns:**
- `gene`: Gene name
- `refseq_id`: RefSeq accession
- `rpf_cds`, `rna_cds`: Read counts
- `TE`, `log2_TE`: Translation efficiency values
- `utr5_seq`: 5' UTR sequence (feature)
- `utr5_length`: Sequence length

---

## Key Concepts Summary

### 1. Why use two-strategy approach for 5' UTR?

**Strategy hierarchy:**
1. **Prefer explicit annotations** when available (most accurate)
2. **Infer from CDS start** as fallback (works for most cases)

GenBank records don't always have explicit `5'UTR` annotations, but they always have `CDS` annotations. Since mRNA structure is:
```
[5' UTR] → [CDS] → [3' UTR]
```
We can infer: **5' UTR = sequence[0 : CDS_start]**

This two-strategy approach ensures:
- Maximum accuracy when explicit annotations exist
- Broad coverage when they don't (which is most of the time in this dataset)

### 2. Why log₂(TE)?
- TE values have wide dynamic range (0.025 to 99.6)
- Log transformation:
  - Compresses range to more manageable scale
  - Makes distribution more symmetric (better for modeling)
  - TE=1 → log₂(TE)=0 (baseline)
  - TE=2 → log₂(TE)=1 (2× more translated)
  - TE=0.5 → log₂(TE)=-1 (2× less translated)

### 3. Rate Limiting
NCBI enforces limits to prevent server overload:
- 3 requests/second without API key
- 10 requests/second with API key
- Exceeding limits → temporary IP ban
- Solution: `time.sleep(0.34)` between requests

---

## Next Steps with the Data

### Machine Learning Approaches

1. **One-hot encoding**:
   ```python
   # Convert DNA sequence to numerical matrix
   # A=[1,0,0,0], C=[0,1,0,0], G=[0,0,1,0], T=[0,0,0,1]
   ```

2. **K-mer features**:
   ```python
   # Extract all 3-mers, 4-mers, etc.
   # "ATCG" → ["ATC", "TCG"]
   ```

3. **Convolutional Neural Networks (CNNs)**:
   - Good at detecting sequence motifs (like Kozak sequence)
   - Learns position-invariant features

4. **Transformer models**:
   - Can learn long-range dependencies
   - Similar to BERT for text, but for DNA sequences

### Example Model Structure
```python
# Input: 5' UTR sequence (variable length)
# Output: Predicted TE (continuous value) or High/Low class

# Possible architecture:
# 1. Embedding layer (convert nucleotides to vectors)
# 2. Convolutional layers (detect motifs)
# 3. Pooling (aggregate information)
# 4. Dense layers (final prediction)
```

This dataset provides ~16,000+ labeled examples for training such models!
