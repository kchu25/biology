@def title = "Processing Ribosome Profiling Data for Translation Efficiency"
@def published = "12 January 2026"
@def tags = ["rna", "screening"]

# Processing Ribosome Profiling Data for Translation Efficiency

## What You're Working With

You have **ribosome profiling (Ribo-seq)** and **RNA-seq** data. The key insight:

- **Ribo-seq** = how much is being translated (ribosomes on mRNA)
- **RNA-seq** = how much mRNA is present
- **Translation Efficiency (TE)** = Ribo-seq / RNA-seq

## The Data Structure

Each sample file has counts mapped to different transcript regions:
- 5' leader (5' UTR)
- CDS (coding sequence)
- 3' UTR

For example, TCOF1 with transcript NM_000356 has:
- 0 reads in 5' UTR
- 369 reads in CDS
- 1 read in 3' UTR

## Step-by-Step Processing

### 1. **Calculate Translation Efficiency**

For each gene/transcript, compute:

$$\text{TE} = \frac{\text{Ribo-seq reads (CDS)}}{\text{mRNA-seq reads (CDS)}}$$

**Why CDS?** That's where active translation happens. The 5' UTR influences translation but isn't translated itself.

**Practical steps:**
- Match control riboseq replicate 1 with control mRNAseq replicate 1
- For each RefSeq ID, divide riboseq CDS counts by mRNAseq CDS counts
- Filter out low-count genes (e.g., require ≥10 reads in both)

### 2. **Get the Sequences**

You need the actual nucleotide sequences as features. Two key regions:

**5' UTR sequence** → This is your primary feature for predicting TE
- Download from NCBI using the RefSeq accession (e.g., NM_000356)
- Extract the 5' UTR region specifically

**CDS start context** → Often helpful too
- Include ~50-100 nt around the start codon

### 3. **Obtain Sequences from NCBI**

```python
from Bio import Entrez
Entrez.email = "your.email@example.com"

# For each RefSeq ID in your data
handle = Entrez.efetch(db="nucleotide", 
                       id="NM_000356", 
                       rettype="gb", 
                       retmode="text")
record = SeqIO.read(handle, "genbank")
```

Parse the GenBank features to extract 5' UTR coordinates.

### 4. **Build Your Dataset**

Each training example:
- **Input (X)**: 5' UTR sequence (or 5' UTR + start codon context)
- **Label (y)**: Translation efficiency (TE value)

**Encoding options:**
- One-hot encoding: A=[1,0,0,0], C=[0,1,0,0], etc.
- k-mer embeddings
- Learned embeddings (like in NLP)

### 5. **Normalization Considerations**

Raw TE values can be noisy. Common approaches:

$$\text{TE}_{\text{norm}} = \frac{\text{Ribo-seq}_{\text{gene}} / \text{Ribo-seq}_{\text{total}}}{\text{mRNA-seq}_{\text{gene}} / \text{mRNA-seq}_{\text{total}}}$$

Or use log₂(TE) to handle the wide dynamic range.

### 6. **Handle Replicates**

You have 2 biological replicates. Options:
- **Average** the TE across replicates (more stable labels)
- **Keep separate** (more training data, but correlated)
- Use one for training, one for validation

### 7. **Filter Your Data**

Important quality controls:
- Remove genes with 0 mRNA-seq reads (undefined TE)
- Filter low-count genes (< 10 reads) → noisy estimates
- Remove outliers (TE > 100 or < 0.01 are suspicious)
- Consider only protein-coding transcripts initially

## Quick Workflow Summary

1. **Download all 8 sample files** from GEO
2. **Pair up** riboseq and mRNAseq samples (control rep1 + control rep1, etc.)
3. **Calculate TE** for each RefSeq transcript
4. **Fetch sequences** from NCBI for each RefSeq ID
5. **Extract 5' UTRs** from GenBank annotations
6. **Create dataset**: (sequence → TE value) pairs
7. **Split** into train/validation/test sets

## Why This Matters

The 5' UTR contains regulatory elements that control translation:
- **Kozak sequence** around start codon
- **Secondary structures** that block ribosomes
- **uORFs** (upstream open reading frames)
- **IRES** elements

Your model will learn which sequence features predict high/low translation efficiency!

## Arsenal vs Control Note

You have both control and arsenite-treated samples. Start with **control samples only** for cleaner biology. Arsenite induces stress granules and affects translation globally—you can explore this later for stress-response prediction.