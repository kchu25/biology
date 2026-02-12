@def title = "Biology Behind the Data: Rosenberg et al., Cell 2015"
@def published = "12 February 2026"
@def tags = ["rna", "screening"]

# Biology Behind the Data: Rosenberg et al., Cell 2015

> **Paper:** *Learning the Sequence Determinants of Alternative Splicing from Millions of Random Sequences*
> Rosenberg, Patwardhan, Shendure & Seelig, Cell 163(3), 2015.
>
> **TL;DR for CS folks:** They built two massive synthetic DNA libraries (~2M and ~265K sequences) with random regions, put them into human cells, measured how each sequence gets spliced (a continuous ratio between 0 and 1), and trained a model to predict splicing from sequence alone.

---

## Table of Contents

1. [Motivation: Why Care About Splicing?](#1-motivation-why-care-about-splicing)
2. [Background: What Is Splicing?](#2-background-what-is-splicing)
3. [The Experimental Design (Big Picture)](#3-the-experimental-design-big-picture)
4. [The Two Libraries: A5SS and A3SS](#4-the-two-libraries-a5ss-and-a3ss)
5. [Pipeline: From DNA to Data](#5-pipeline-from-dna-to-data)
6. [How Labels Are Computed](#6-how-labels-are-computed)
7. [What's In the CSV Files](#7-whats-in-the-csv-files)
8. [The Model (Briefly)](#8-the-model-briefly)
9. [Summary Table](#9-summary-table)

---

## 1. Motivation: Why Care About Splicing?

In humans, most genes are not contiguous blocks of code. They are split into **exons** (the parts that end up in the final mRNA) and **introns** (the parts that get removed). The process of removing introns and stitching exons together is called **splicing**.

Here's the kicker: the cell doesn't always splice the same way. It can choose to include or skip certain exons, or use different splice boundaries. This is called **alternative splicing**, and it's how ~20,000 human genes produce >100,000 distinct protein variants.

```
The "splicing code" problem:
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Given a DNA sequence around a splice site,
can we predict HOW the cell will splice it?

    Input:  DNA sequence (string of A/C/G/T)
    Output: Splicing ratio (float between 0 and 1)
```

**Why it matters:**
- ~15–50% of disease-causing mutations disrupt splicing
- Understanding the sequence → splicing relationship = understanding gene regulation
- Prior work relied on natural sequences (confounded by evolution, limited diversity)

**What Rosenberg et al. did differently:** Instead of studying the ~thousands of natural alternative splice events in the human genome, they **synthesized millions of random sequences** and measured splicing for each one. Random sequences are unbiased — they explore sequence space uniformly, which is ideal for learning generalizable rules.

> **CS analogy:** Imagine you want to learn a function `f(x) → y`. Prior work sampled `x` from a narrow, correlated distribution (natural genomes). This paper generates `x` uniformly at random and measures `y` for each — a much better training set.

---

## 2. Background: What Is Splicing?

If you've never seen this before, here's the 60-second version:

```
DNA (in the genome):

    ═══[EXON 1]════════════[INTRON]════════════[EXON 2]═══
                  ↑                        ↑
            splice donor (SD)        splice acceptor (SA)
              "GT"                       "AG"
              (5' splice site)           (3' splice site)

After splicing (mRNA):

    ═══[EXON 1]═══[EXON 2]═══     +     ⌒ (discarded intron)
```

**Key signals the cell uses to decide where to splice:**

| Signal | What it is | Where |
|--------|-----------|-------|
| **Splice donor (SD)** | Almost always starts with `GT` | 5' end of intron |
| **Splice acceptor (SA)** | Almost always ends with `AG` | 3' end of intron |
| **Branch point** | An `A` nucleotide upstream of SA | Inside intron |
| **Polypyrimidine tract** | Run of C/T before the SA | Inside intron |
| **ESE** (Exonic Splicing Enhancer) | Short motifs in exons | Exons |
| **ESS** (Exonic Splicing Silencer) | Short motifs in exons | Exons |
| **ISE** (Intronic Splicing Enhancer) | Short motifs in introns | Introns |
| **ISS** (Intronic Splicing Silencer) | Short motifs in introns | Introns |

The cell's **spliceosome** (a massive RNA-protein machine) reads these signals and decides where to cut. When there are **two competing splice sites**, the spliceosome must choose between them — and the surrounding sequence determines the ratio.

---

## 3. The Experimental Design (Big Picture)

This is a **Massively Parallel Reporter Assay (MPRA)** — the same family of experiments described in [this blog post](https://kchu25.github.io/biology/blog/general/mpra/).

### The Core Pattern

```
Input  →  Instrument  →  Measurement

DNA sequence  →  Human cells (splicing machinery)  →  Splice site usage ratio
```

### What makes this an MPRA:

1. **Massively parallel**: Hundreds of thousands of sequences tested simultaneously in one experiment
2. **Reporter**: A fluorescent protein (Citrine) serves as the readout scaffold
3. **Assay**: Sequencing-based measurement of splice site usage

### High-Level Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIBRARY CONSTRUCTION                         │
│                                                                 │
│  1. Design a reporter construct (Citrine split by an intron)    │
│  2. Synthesize oligos with RANDOM intronic/exonic regions       │
│  3. Clone into plasmid (each gets a unique barcode)             │
│  4. Integrate ONE copy per cell (FLP-FRT system)                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DNA SEQUENCING                                │
│                                                                 │
│  Purpose: Build a lookup table of barcode → sequence            │
│                                                                 │
│  • Read 1 = random intronic/exonic region (the variable part)   │
│  • Read 2 = barcode in 3'UTR (unique ID for each construct)    │
│                                                                 │
│  Result: "Barcode X belongs to sequence Y"                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RNA SEQUENCING                                │
│                                                                 │
│  Purpose: Measure which splice site was used for each barcode   │
│                                                                 │
│  • Read 1 = exon-exon junction (reveals WHERE splicing happened)│
│  • Read 2 = barcode (reveals WHICH sequence we're measuring)    │
│                                                                 │
│  Result: "Barcode X used splice site SD1 this many times,       │
│           and splice site SD2 this many times"                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMPUTE LABELS                                │
│                                                                 │
│  For each sequence, compute:                                    │
│                                                                 │
│        label = (reads at alternative site)                      │
│                ─────────────────────────────                    │
│                (reads at ALL splice sites)                       │
│                                                                 │
│  Result: A number between 0 and 1 for each sequence            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. The Two Libraries: A5SS and A3SS

The paper builds two separate libraries, each testing a different type of alternative splicing:

### A5SS: Alternative 5' Splice Site

"5' splice site" = splice **donor** (the GT that marks where the intron begins).

In this library, there are **two competing splice donors** (SD1 and SD2), and the cell must choose between them.

```
A5SS Construct Layout (inside the intron):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ═══[Citrine Exon 1]═╤══════════[INTRON: 303 bp]══════════╤═[Citrine Exon 2]═══
                       │                                    │
                      SD1                                   SA
                  (constitutive,                        (sole acceptor)
                   always GT)

  Zoom into the intron:
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │  pos 0      pos 7        pos 31  pos 44      pos 50       pos 74        │
  │   │          │              │      │           │             │            │
  │   SD1──┤ R1 (25nt random) ├──────SD2──┤ R2 (25nt random) ├────...──SA  │
  │   GT   └──── intronic ────┘       GT   └──── intronic ────┘             │
  │  (always                      (competing                                │
  │   present)                     donor)                                   │
  │                                                                          │
  └──────────────────────────────────────────────────────────────────────────┘

  What the cell sees:
  • SD1 (position 0):  constitutive donor — always available
  • SD2 (position 44): alternative donor — in the random region
  • The 25nt random regions (R1, R2) contain unknown regulatory motifs
  • The cell's spliceosome "decides" how often to use SD1 vs SD2
```

**What varies:** R1 (25 random nucleotides, intronic) and R2 (25 random nucleotides, intronic). Everything else is fixed.

**What's in the CSV:** A 101-nucleotide sequence — the intron from SD1 to the splice acceptor, including both random regions and the fixed flanking context:

```
101 nt = TGCTTGG[R1: 25nt]GGTCGACCCAGGTTCGTG[R2: 25nt]GAGGTATTCTTATCACCTTCGTGGCT
         ───────              ──────────────────              ─────────────────────────
         fixed                fixed                           fixed
                  ──────────                    ──────────
                   random                        random
```

### A3SS: Alternative 3' Splice Site

"3' splice site" = splice **acceptor** (the AG that marks where the intron ends).

In this library, there are **two competing splice acceptors** (SA1 and SA2), and the cell must choose between them.

```
A3SS Construct Layout (inside the intron):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ═══[Citrine Exon 1]═╤══════════[INTRON: 388 bp]══════════╤═[Citrine Exon 2]═══
                       │                                    │
                       SD                                  SA2
                  (sole donor)                         (constitutive,
                                                        always AG)

  Zoom into the intron (near the 3' end):
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │  ...──┤ R1 (25nt random) ├──────SA1──┤ R2 (25nt random) ├────SA2       │
  │        └──── intronic ────┘      AG   └──── exonic!! ────┘     AG       │
  │                               (alt.                          (const.)   │
  │                              acceptor)                                  │
  │                                                                          │
  └──────────────────────────────────────────────────────────────────────────┘

  Key difference from A5SS:
  • R1 is INTRONIC (before the alternative acceptor SA1)
  • R2 is EXONIC (between SA1 and SA2 — if SA1 is used, R2 becomes part of the exon!)
```

**What varies:** R1 (25 random nucleotides, intronic) and R2 (25 random nucleotides, exonic). Everything else is fixed.

**What's in the CSV:** An 80-nucleotide sequence spanning R1, the SA1 site, and R2:

```
80 nt = [R1: 25nt]GCTTGGATCTGATCTCAACAGGGT[R2: 25nt]ATGATT
                   ────────────────────────            ──────
                   fixed (contains SA1=AG)             fixed
        ──────────                          ──────────
         random                              random
```

### Why A3SS labels are so small

SA2 (the constitutive acceptor) is very strong — it has a nearly perfect splice acceptor signal. SA1 (the alternative acceptor, which sits in a random context) almost never wins the competition. So most sequences have a label ≈ 0 (SA2 dominates). The mean label is ~0.016. This is **biologically correct**, not a bug.

---

## 5. Pipeline: From DNA to Data

Here is the full experimental pipeline, step by step:

### Step 1: Oligo Synthesis

```
┌───────────────────────────────────────────────────────────┐
│  Microarray-synthesized oligonucleotide pool              │
│                                                           │
│  Each oligo: [fixed flank][R1: 25nt random][fixed][R2: 25nt random][fixed flank]  │
│                                                           │
│  A5SS: ~265,000 unique sequences                          │
│  A3SS: ~2,200,000 unique sequences                        │
│                                                           │
│  The "random" regions are synthesized with equal           │
│  probability of A, C, G, T at each position.              │
│  4^50 possible combinations ≈ 1.27 × 10^30               │
│  (we only sample a tiny fraction)                         │
└───────────────────────────────────────────────────────────┘
```

### Step 2: Cloning Into Plasmid

```
┌───────────────────────────────────────────────────────────┐
│  Plasmid backbone (~8.4–8.5 kb):                          │
│                                                           │
│  • EF-1α promoter (drives transcription)                  │
│  • Citrine (yellow fluorescent protein, split by intron)  │
│  • Synthetic intron with the random regions               │
│  • 20-nt barcode in the 3'UTR (unique per construct)     │
│  • FRT sites (for single-copy genomic integration)        │
│  • Hygromycin resistance (for selecting integrated cells) │
│  • mCherry on the opposite strand (integration control)   │
│                                                           │
│  Result: each plasmid = one unique sequence + one barcode  │
└───────────────────────────────────────────────────────────┘
```

### Step 3: Genomic Integration (FLP-FRT)

```
┌───────────────────────────────────────────────────────────┐
│  HEK293 cells with a single FRT "landing pad"             │
│                                                           │
│  FLP recombinase inserts EXACTLY ONE copy of the          │
│  reporter construct into a defined genomic location.      │
│                                                           │
│  Why single-copy?                                         │
│  • No copy-number variation confounding the measurement   │
│  • Every cell expresses exactly one variant               │
│  • Controlled chromatin context                           │
│                                                           │
│  Hygromycin selection → only integrated cells survive     │
│  (Pool of cells, each with a different sequence)          │
└───────────────────────────────────────────────────────────┘
```

### Step 4: DNA-seq (Barcode → Sequence Mapping)

```
┌───────────────────────────────────────────────────────────┐
│  Extract genomic DNA from the cell pool                   │
│  PCR amplify the reporter region                          │
│  Paired-end Illumina sequencing:                          │
│                                                           │
│     Read 1 ──→ [random intronic/exonic region]            │
│     Read 2 ──→ [20-nt barcode in 3'UTR]                 │
│                                                           │
│  Result: a dictionary                                     │
│     barcode_to_sequence = {                               │
│         "ACGT...TAGC": "NNNN...NNN",  # 20-nt → seq     │
│         "TGCA...GATC": "NNNN...NNN",                     │
│         ...                                               │
│     }                                                     │
│                                                           │
│  This is conceptually a JOIN key for the RNA-seq step.    │
└───────────────────────────────────────────────────────────┘
```

### Step 5: RNA-seq (Measuring Splicing)

```
┌───────────────────────────────────────────────────────────┐
│  Extract total RNA from the cell pool                     │
│  Reverse transcribe → cDNA                                │
│  PCR amplify across the exon-exon junction                │
│  Paired-end Illumina sequencing:                          │
│                                                           │
│     Read 1 ──→ [exon-exon junction]                      │
│                  ↑                                        │
│                  This tells you WHERE the splice happened │
│                  (position of the junction = which SD/SA  │
│                   was used)                               │
│                                                           │
│     Read 2 ──→ [20-nt barcode]                           │
│                  ↑                                        │
│                  This tells you WHICH sequence was in     │
│                  the cell that produced this mRNA         │
│                                                           │
│  Result: a count matrix                                   │
│     rows = sequences (identified by barcode)              │
│     columns = splice junction positions                   │
│     values = number of reads supporting that junction     │
└───────────────────────────────────────────────────────────┘
```

### Step 6: Compute Labels

```
┌───────────────────────────────────────────────────────────┐
│  The count matrix is stored in Reads.mat (sparse matrix)  │
│                                                           │
│  A5SS matrix: 265,137 rows × 304 columns                 │
│  A3SS matrix: 2,211,739 rows × 606 columns               │
│                                                           │
│  Each column = a specific exon-exon junction position     │
│  Each row = one sequence variant                          │
│  Each value = number of RNA-seq reads at that junction    │
│                                                           │
│  Label computation:                                       │
│                                                           │
│    A5SS:  label = SD2_reads / (SD1_reads + SD2_reads)     │
│           where SD1 = column 0, SD2 = column 44           │
│                                                           │
│    A3SS:  label = SA1_reads / total_reads                 │
│           where SA1 = column 235                          │
│           total = sum of all junction reads for that row  │
│                                                           │
│  Filter: only keep sequences with ≥ 10 total reads       │
│  (discard low-coverage sequences for statistical power)   │
└───────────────────────────────────────────────────────────┘
```

---

## 6. How Labels Are Computed

### The Measurement

| Property | Value |
|----------|-------|
| **What's measured** | Ratio of RNA-seq reads at competing splice sites |
| **Unit** | Dimensionless ratio (0 to 1) |
| **Analogous to** | PSI (Percent Spliced In), a standard metric in splicing biology |
| **0 means** | The alternative site is NEVER used (constitutive site wins 100%) |
| **1 means** | The alternative site is ALWAYS used (alternative site wins 100%) |
| **0.5 means** | Both sites are used equally |

### A5SS Label Formula

```
                    reads mapping to SD2 junction
label_A5SS = ──────────────────────────────────────────────
             reads mapping to SD1 junction + reads mapping to SD2 junction

           =  Reads.mat[row, column 44]
              ─────────────────────────────────────────
              Reads.mat[row, column 0] + Reads.mat[row, column 44]
```

- **SD1** (column 0): The constitutive splice donor at position 0. This is "always there."
- **SD2** (column 44): The alternative splice donor at position 44. This is the one whose usage we're measuring.
- A label of 0.68 (the dataset mean) means the alternative donor SD2 is used 68% of the time on average — which means the random sequences often create contexts that activate this site.

### A3SS Label Formula

```
                    reads mapping to SA1 junction
label_A3SS = ──────────────────────────────────────────────
                    total reads across ALL junctions

           =  Reads.mat[row, column 235]
              ─────────────────────────────────
              sum(Reads.mat[row, :])
```

- **SA1** (column 235): The alternative splice acceptor.
- **SA2** (column 388): The constitutive (dominant) splice acceptor.
- A label of 0.016 (the dataset mean) means SA1 is used only ~1.6% of the time — SA2 almost always wins.

### Why Use Ratios?

Ratios normalize out:
- Different expression levels between constructs
- Different amounts of total RNA per cell
- PCR amplification biases

This gives you a clean measurement of **splice site preference** independent of expression level.

---

## 7. What's In the CSV Files

### `extracted/A5SS_seq2label.csv`

| Column | Description | Example |
|--------|------------|---------|
| `sequence` | 101-nt DNA sequence (intronic region from SD1 to SA) | `TGCTTGG...TGGCT` |
| `label` | SD2/(SD1+SD2) ratio | `0.7234` |
| `total_reads` | SD1 + SD2 read count | `42` |
| `sd1_reads` | Reads at the constitutive donor (column 0) | `12` |
| `sd2_reads` | Reads at the alternative donor (column 44) | `30` |

**Stats:** 251,153 sequences, mean label = 0.676, std = 0.380

### `extracted/A3SS_seq2label.csv`

| Column | Description | Example |
|--------|------------|---------|
| `sequence` | 80-nt DNA sequence (R1 + fixed + R2 region) | `AACTG...TGATT` |
| `label` | SA1/total ratio | `0.0023` |
| `total_reads` | Sum of all junction reads | `156` |
| `sa1_reads` | Reads at the alternative acceptor (column 235) | `0` |

**Stats:** 37,299 sequences, mean label = 0.016, std = 0.069

---

## 8. The Model (Briefly)

The authors train a **Multinomial Logistic Regression (MLR)** model:

```
Features:
  • 6-mer frequencies in 8 defined regions of the sequence
  • Regions: segments of the intron/exon around splice sites
  • 4^6 = 4,096 possible 6-mers × 8 regions = 32,768 features

Loss: KL divergence between predicted and observed splice ratios
Regularization: L1 (Lasso) — encourages sparsity in the 6-mer weights

Interpretation:
  • Large positive weight for a 6-mer in a region → that motif INCREASES
    use of the alternative splice site in that context
  • Large negative weight → that motif DECREASES alternative site usage
  • Known splicing motifs (ESEs, ESSs, ISEs, ISSs) emerge from the data
```

---

## 9. Summary Table

| Aspect | A5SS Library | A3SS Library |
|--------|-------------|-------------|
| **What's tested** | Alternative 5' splice site (donor) | Alternative 3' splice site (acceptor) |
| **Competing sites** | SD1 (constitutive) vs SD2 (alternative) | SA1 (alternative) vs SA2 (constitutive) |
| **Random regions** | R1: 25nt intronic, R2: 25nt intronic | R1: 25nt intronic, R2: 25nt exonic |
| **Sequence in CSV** | 101 nt | 80 nt |
| **# sequences (raw)** | 265,137 | 2,211,739 |
| **# sequences (filtered, ≥10 reads)** | 251,153 | 37,299 |
| **Label formula** | SD2 / (SD1 + SD2) | SA1 / total |
| **Label unit** | Dimensionless ratio [0, 1] | Dimensionless ratio [0, 1] |
| **Mean label** | 0.676 | 0.016 |
| **Interpretation of mean** | Alt. donor used ~68% of the time | Alt. acceptor used ~1.6% of the time |
| **Reads.mat columns used** | 0 (SD1), 44 (SD2) | 235 (SA1), all (total) |
| **Reporter** | Citrine (yellow fluorescent protein) | Citrine (yellow fluorescent protein) |
| **Integration** | FLP-FRT single-copy | FLP-FRT single-copy |
| **Cell line** | HEK293 | HEK293 |

---

## Glossary for CS People

| Biology Term | CS Translation |
|-------------|---------------|
| **Gene** | A "function definition" in the genome that produces a protein |
| **Exon** | The lines of code that make it into the compiled output |
| **Intron** | Comments / dead code that gets removed before compilation |
| **Splicing** | The "compiler" step that removes introns and joins exons |
| **Alternative splicing** | The compiler non-deterministically choosing different compilation paths |
| **Splice donor (GT)** | Start delimiter for an intron |
| **Splice acceptor (AG)** | End delimiter for an intron |
| **Spliceosome** | The compiler itself (a complex molecular machine) |
| **MPRA** | A/B testing at scale — millions of variants tested simultaneously |
| **Barcode** | A unique hash/ID tag for each sequence variant |
| **PSI / splicing ratio** | The measured "probability" of taking one compilation path vs another |
| **6-mer** | A substring of length 6 (feature for the model) |

---

## References

- Rosenberg AB, Patwardhan RP, Shendure J, Seelig G. **Learning the Sequence Determinants of Alternative Splicing from Millions of Random Sequences.** *Cell* 163(3):698-711, 2015. [DOI: 10.1016/j.cell.2015.09.054](https://doi.org/10.1016/j.cell.2015.09.054)
- Repository: [Alex-Rosenberg/cell-2015](https://github.com/Alex-Rosenberg/cell-2015)
- MPRA overview: [kchu25.github.io/biology/blog/general/mpra/](https://kchu25.github.io/biology/blog/general/mpra/)
