@def title = "Johns et al. (2018) E. coli Expression Units Explained"
@def published = "19 January 2026"
@def tags = ["data-processing"]

# Johns et al. (2018) E. coli Expression Units Explained

## YES - Your Data is log₂(RNA/DNA)

Looking at your data format:
```
seq,expr
TCCCACTATTTGTCGGCTAGCCAGATTGTTGTGGTCTGATTAAAGTTTCAATTTATACCTTACAATGATGTAAGGTACGTGTAAGAGAAATCGATGGGATATTTTTTTACAACAAGGTATTCTTAAAGTAAGAGTTATACGCTATGTGGAAAAGAGGTGTTTAAG,2.346474234
```

The `expr` column = **log₂(RNA/DNA)**, a normalized transcription measurement.

## The Measurement Unit

**Unit:** log₂(RNA/DNA) normalized by control sequences

### What This Means

$
\text{Expression} = \log_2\left(\frac{\text{RNA reads}}{\text{DNA reads}}\right) - \text{control mean}
$

**Where:**
- **RNA reads** = mRNA molecule counts from RNA-seq of the pooled library
- **DNA reads** = DNA barcode counts from DNA-seq of the same library
- **Control mean** = average log₂(RNA/DNA) of control sequences (for normalization)

### Why RNA/DNA?

The DNA sequencing tells you how many cells have each promoter construct:
- If promoter X appears 1,000 times in DNA-seq → 1,000 cells have it
- If it appears 5,000 times in RNA-seq → high transcription
- Ratio = 5,000/1,000 = 5 → log₂(5) = 2.32

The ratio **normalizes for**:
- Different representation of promoters in the library
- Sequencing depth variations
- Number of cells with each construct

## Your Data Values

Looking at your examples:
- `2.346` = moderate transcription (2^2.35 ≈ 5× more RNA than DNA)
- `5.566` = high transcription (2^5.57 ≈ 47× more RNA than DNA)
- `-2.138` = low/repressed (2^-2.14 ≈ 0.23× RNA/DNA, less RNA than DNA)

Negative values mean **repression** - less RNA was made than you'd expect from the DNA counts.

## Where in the Paper (Specific References)

**Figure 2a caption:**
> "Transcriptional activity of 11,319 RSs measured in B. subtilis, E. coli, and P. aeruginosa, shown as **log2 (RNA/DNA) ratios normalized by the mean activity of control sequences**"

**Supplementary Figure 7a caption:**
> "Transcription levels are **log2 (RNA/DNA) ratios normalized by the mean activity of control sequences** (see Methods)"

**Methods section** (in supplementary materials):
> "Transcription activity was quantified as log₂(RNA/DNA)"

**Figure 1 (schematic):**
> Shows "Targeted RNA-seq, DNA-seq, and FACS-seq enable accurate multiplexed measurement of transcription and translation levels"

## The Complete Pipeline

```
1. Pool of bacteria with different promoters
       ↓
2. Extract RNA → RNA-seq → count mRNA for each promoter
       ↓
3. Extract DNA → DNA-seq → count DNA barcode for each promoter  
       ↓
4. Calculate: log₂(RNA count / DNA count)
       ↓
5. Normalize by subtracting mean of control sequences
       ↓
6. Final value in your "expr" column
```

## Translation Data (NOT in your file)

They ALSO measured translation using **FACS-seq** (flow cytometry of GFP), but that's separate data. Your file only has the **transcription** measurements (log₂(RNA/DNA)).

## Bottom Line

**Your `expr` column = log₂(RNA/DNA) - control_mean**

- **Unit:** dimensionless, log-scaled ratio
- **Measures:** transcriptional activity (how much mRNA is produced)
- **NOT:** protein level, absolute molecule counts, or GFP fluorescence
- **Range:** typically -3 to +6 (negative = repressed, positive = active)