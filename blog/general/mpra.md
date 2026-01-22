@def title = "MPRA Pipeline: From DNA to Measurements"
@def published = "22 January 2026"
@def tags = ["experimental-biology"]

# MPRA Pipeline: From DNA to Measurements

Looking at these studies, there's a common experimental thread—they're all **massively parallel reporter assays (MPRAs)**. Let me break down what's actually happening.

## The Core Pattern: Input → Instrument → Measurement

Think of it like this: you're testing hundreds of thousands of DNA sequences to see what they *do*. The input is always **a DNA sequence** (promoter, 5' UTR, or 3' UTR), and the measurement is some proxy for **gene expression activity**.

### Input: The DNA Sequence Library
- **Promoters** (de Boer): Random DNA sequences to test transcription initiation
- **5' UTRs** (Sample, Cuperus, Cao): 50-nt sequences controlling translation
- **3' UTRs** (Rabani): Sequences controlling mRNA stability
- Typically 10,000 - 500,000 unique variants per experiment

### The Measurement: What Are We Tracking?

Each study measures a different layer of gene expression:

| Study | What's Measured | Units |
|-------|----------------|-------|
| **de Boer (yeast)** | Transcription strength | log₂(YFP/RFP fluorescence ratio) |
| **Johns (bacteria)** | Transcription activity | log₂(RNA reads / DNA reads) |
| **Cuperus (yeast)** | Translation efficiency + growth | log₂(enrichment after selection) |
| **Sample (human)** | Ribosome loading | Mean Ribosome Load (0-8+ ribosomes) |
| **Cao (human)** | Translation efficiency | log₂(enrichment), then % of control |
| **Rabani (zebrafish)** | mRNA stability over time | log₂(mRNA count / initial count) |

Notice the pattern? **Almost everything is a log₂ ratio.** Why? Because gene expression spans orders of magnitude (a weak promoter might be 100× weaker than a strong one), and ratios naturally cancel out confounding variables.

## The Instrument Pipeline

Here's where it gets interesting. These aren't single-measurement experiments—they're sophisticated pipelines:

```
┌─────────────────────┐
│  DNA Library        │  ← 10K-500K unique sequences
│  (Synthesized)      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Cloning into       │  ← Put each sequence into a plasmid
│  Reporter Construct │     with a fluorescent protein or
└──────────┬──────────┘     survival gene
           │
           ▼
┌─────────────────────┐
│  Transformation     │  ← Get DNA into cells
│  into Cells         │     (bacteria, yeast, mammalian)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Expression Period  │  ← Let cells express the gene
└──────────┬──────────┘     (hours to days)
           │
           ├─────────────────────────────────────┐
           │                                     │
           ▼                                     ▼
┌─────────────────────┐              ┌─────────────────────┐
│  Flow Cytometry     │              │  Next-Gen           │
│  (FACS)             │              │  Sequencing         │
│                     │              │                     │
│  • Measures fluor-  │              │  • RNA-seq          │
│    escence per cell │              │  • DNA-seq          │
│  • Sorts cells into │              │  • Count reads per  │
│    bins             │              │    variant          │
│  • 10K-400K cells/s │              │  • Quantifies       │
└──────────┬──────────┘              │    abundance        │
           │                         └──────────┬──────────┘
           │                                    │
           └──────────┬─────────────────────────┘
                      ▼
           ┌─────────────────────┐
           │  Deep Sequencing    │  ← Identify which sequences
           │  of Sorted Pools    │     were in which bin
           └──────────┬──────────┘
                      │
                      ▼
           ┌─────────────────────┐
           │  Read Counting &    │  ← Map sequence → activity
           │  Normalization      │
           └──────────┬──────────┘
                      │
                      ▼
           ┌─────────────────────┐
           │  Final Measurement  │  ← log₂(ratio) or enrichment
           └─────────────────────┘
```

## Key Instruments at Each Stage

**1. DNA Synthesis & Cloning**
- Oligonucleotide synthesizers (or order from Twist, IDT)
- Gibson assembly or restriction cloning
- Transformation (electroporation or heat shock)

**2. The Expression Phase**
Different approaches depending on what you're measuring:
- **Flow cytometry approach**: Fluorescent proteins (YFP/RFP, GFP/BFP)
- **Sequencing approach**: Extract RNA and DNA from pooled cells
- **Selection approach**: Grow cells and see which variants survive
- **Polysome profiling**: Separate mRNA by ribosome occupancy

**3. Sorting & Separation**
- **Flow cytometer (FACS)**: Laser-based cell sorter, 10K-400K cells/second
  - Used in: de Boer, Cao, Sample (for validation)
- **Sucrose gradient ultracentrifuge**: Separates by density
  - Used in: Sample (polysome profiling)
- **Growth selection**: Natural selection in liquid culture
  - Used in: Cuperus

**4. Sequencing**
- **Illumina sequencer**: Gold standard, millions of reads
  - Used in: all studies
- Counts how many times each DNA sequence appears
- Typical output: 10-100 million reads per sample

**5. Normalization**
This is the secret sauce. The measurement is always a **ratio** to cancel out:
- Copy number variation (plasmids per cell)
- Sequencing depth differences
- PCR amplification bias
- Cell-to-cell heterogeneity

## The Normalization Trick

**Promoter studies** (de Boer):
- Measure both YFP (test promoter) and RFP (reference promoter)
- Ratio cancels out: plasmid copy number, cell size, detector sensitivity

**Transcription studies** (Johns):
- RNA-seq / DNA-seq ratio
- Cancels out: library representation, sequencing depth

**Translation studies** (Cuperus, Sample, Cao):
- Before/after enrichment OR ribosome occupancy
- Cancels out: PCR bias, initial abundance

**Stability studies** (Rabani):
- Normalized to initial count + spike-in controls
- Cancels out: sequencing depth, extraction efficiency

## Why This Design Is Brilliant

By pooling hundreds of thousands of variants and measuring them simultaneously:
- **Throughput**: Test 500K sequences in one experiment
- **Controlled conditions**: All variants compete in the same environment
- **Cost-effective**: One sequencing run, massive data
- **Internal controls**: Variants normalize against each other

The real innovation isn't just high-throughput—it's the ability to **decouple variables**. By measuring multiple things (DNA + RNA, or YFP + RFP, or before + after), you can isolate the signal you care about from all the noise.

## From Wet Lab to Data

The final output is always a table:
```
sequence              measurement
ATCG...TGCA          2.35
GGTA...CCAA          -1.42
TCCC...TAAG          5.67
```

That measurement column—whether it's log₂(YFP/RFP), log₂(RNA/DNA), enrichment score, or mean ribosome load—represents the **biological activity** of that sequence, cleaned of all the technical noise through clever normalization.

Pretty elegant, right? You transform a molecular biology problem into a data science problem, then use CNNs to learn the sequence-function relationship.