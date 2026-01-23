@def title = "MPRA Pipeline: From DNA to Measurements"
@def published = "23 January 2026"
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

### Breaking Down the Measurement Units (What the Numbers Actually Mean)

| Study | Raw Measurement | How It's Normalized | Final Value Type | Interpretation Example |
|-------|----------------|---------------------|------------------|------------------------|
| **de Boer** | YFP fluorescence (AFU), RFP fluorescence (AFU) | log₂(YFP/RFP) | **Dimensionless log₂ ratio** (Range: -3 to +15) | `5.2` = 2^5.2 ≈ 37× more YFP than RFP; `-2.0` = 2^-2 = 0.25× (weak promoter) |
| **Johns** | RNA-seq read counts, DNA-seq read counts | log₂(RNA/DNA) - control_mean | **Dimensionless log₂ ratio** (Range: -3 to +6) | `2.35` = 2^2.35 ≈ 5× more RNA than DNA; `-2.14` = repressed (less RNA than expected) |
| **Cuperus** | **DNA sequencing reads** of the 50-nt UTR region. **Before**: Initial plasmid library (T₀). **After**: Extracted from surviving yeast after 20 generations in -His medium | log₂(reads_after / reads_before) | **log₂ enrichment score** (Range: -5 to +5). Common formats: "log₂ enrichment", "log₂ fold-change", "selection coefficient" | `3.5` = 2^3.5 ≈ 11× enrichment (good growth); `-3.0` = 2^-3 = 0.125× (died off). **What's actually sequenced**: PCR-amplified DNA from the UTR region |
| **Sample** | Sequencing reads in 14 fractions (fraction 1 = 0 ribosomes, fraction 14 = 8+ ribosomes) | Σ(fraction_proportion × ribosome_count) | **Mean Ribosome Load (MRL)** (Range: 2 to 8+). **Units: number of ribosomes** | `2.5` = average of 2-3 ribosomes per mRNA (weak); `7.8` = average of 7-8 ribosomes per mRNA (strong) |
| **Cao (screening)** | NGS reads in top GFP bin, NGS reads in control bin | log₂(top_bin / control_bin) | **log₂ enrichment score** (Range: varies) | `2.0` = 2^2 = 4× enriched in high expressors |
| **Cao (validation)** | GFP fluorescence (MFI), BFP fluorescence (MFI). Compared to pVAX1 control | (GFP/BFP)_test / (GFP/BFP)_control × 100% | **Percentage of control** (Range: 0% to 500%+). **Units: %** | `158%` = 1.58× more protein than control; `250%` = 2.5× more protein |
| **Cao (validation)** | ELISA protein concentration | Protein amount / cell count / time | **Picograms per milliliter**. Normalized: pg/mL per 100k cells/24h. **Units: pg/mL** | `2760 pg/mL` (raw concentration). Then converted to `552%` of control |
| **Rabani** | UMI counts (unique molecular identifiers), Spike-in controls (known RNA amounts) | Calibrated to molecule count, then log₂(count / initial_fitted_count) | **log₂ fold-change from initial** (Range: -6 to +3). **Dimensionless** | `-3.0` = 2^-3 = 12.5% of mRNA remains; `0.0` = at initial level; `+1.0` = 2× more mRNA |

### Key Distinctions

**Type 1: Dimensionless log₂ ratios** (de Boer, Johns, Cuperus screening, Cao screening, Rabani)
- The value is a **logarithm of a dimensionless ratio**
- To get back to fold-change: **2^value**
- Typical range: -5 to +15
- Example: `3.0` means 2³ = 8× more active

**Type 2: Absolute physical units** (Sample MRL, Cao ELISA)
- **Sample MRL**: Literally counting ribosomes—the value IS the number of ribosomes
  - `5.2` means on average 5.2 ribosomes are on each mRNA molecule
  - This is NOT a log scale—it's a direct count!
- **Cao ELISA**: Measuring protein concentration in pg/mL
  - `2760 pg/mL` = 2.76 nanograms per milliliter
  - Often then converted to % of control for easier comparison

**Type 3: Percentages** (Cao validation with fluorescence)
- Normalized to control = 100%
- `158%` means 1.58× the activity of the control plasmid
- This is **linear scale**, not log scale!

### The Critical Difference: Log vs Linear vs Counts

- **log₂ values**: You must exponentiate to get actual fold-change (2^x)
- **MRL values**: Direct count—no transformation needed
- **Percentage values**: Already in intuitive linear scale
- **pg/mL values**: Physical concentration—real-world measurable quantity

When training ML models, these different value types matter! Log-scaled data has different statistical properties than linear counts or percentages.

## How to Report These Measurements

Here's how each study actually reports their units in papers, and what you should write:

| Study | Short Name for Figures | Methods Section Description | Example Figure Caption | Notes |
|-------|------------------------|----------------------------|------------------------|-------|
| **de Boer** | "Expression" or "log₂(YFP/RFP)" | "Promoter activity was measured as the log₂ ratio of YFP to RFP fluorescence intensity per cell, normalized to the TEF2 reference promoter" | "Promoter expression levels (log₂(YFP/RFP)) for 100M random sequences" | The "log₂" is explicit. YFP/RFP makes it clear both colors are measured |
| **Johns** | "Transcription" or "log₂(RNA/DNA)" | "Transcriptional activity was quantified as log₂(RNA/DNA) ratios, where RNA and DNA read counts were obtained from RNA-seq and DNA-seq respectively, normalized by the mean activity of control sequences" | "Transcription levels (log₂(RNA/DNA)) across three bacterial species" | Explicitly state both RNA-seq AND DNA-seq were performed |
| **Cuperus** | "log₂ enrichment" or<br>"Fitness" or<br>"Selection coefficient" | "Translation efficiency was measured as log₂ enrichment, calculated from the ratio of normalized sequencing read counts after 20 generations of selection in -His medium to initial library counts" | "Translation efficiency (log₂ enrichment) for 500K 5' UTR variants" | Don't say "reads after/before"—say "enrichment" or "fold-change" |
| **Sample** | "MRL" (Mean Ribosome Load) | "Translation was quantified by Mean Ribosome Load (MRL), calculated as the weighted average of ribosome occupancy across 14 polysome fractions, where MRL = Σᵢ (fᵢ × rᵢ), with fᵢ being the fraction of reads in fraction i and rᵢ the number of ribosomes" | "Ribosome loading (MRL) for 280K synthetic 5' UTRs" | This is NOT log-scaled! Report as "MRL = 5.2 ribosomes" not "log₂" |
| **Cao (screening)** | "log₂ enrichment" | "Translation efficiency was screened using FACS-seq. Cells were sorted by GFP intensity, and enrichment scores were calculated as log₂(reads in top 2.5% bin / reads in unsorted pool)" | "Enrichment scores (log₂) for 12K synthetic 5' UTRs" | Specify which bins you're comparing (top vs. control) |
| **Cao (validation)** | "Relative expression (%)" or "% of control" | "Relative expression was measured as (GFP/BFP)test / (GFP/BFP)pVAX1 × 100%, where GFP and BFP fluorescence were measured by flow cytometry. For therapeutic proteins, expression was quantified by ELISA (pg/mL per 100,000 cells per 24h) and normalized to pVAX1 = 100%" | "Relative GFP expression (% of pVAX1 control) for top candidate UTRs" | Be explicit about what "100%" means (your control plasmid) |
| **Rabani** | "log₂ fold-change" or<br>"mRNA decay" or<br>"log₂(normalized count)" | "mRNA stability was measured as log₂ fold-change relative to the fitted initial count (X₀) for each reporter, normalized using spike-in controls and calculated as log₂(UMI count / X₀)" | "mRNA decay profiles (log₂ fold-change from initial) across developmental time points" | Clarify that X₀ is fitted per-reporter, not a single global value |

### General Principles for Reporting

**In your figure axes:**
- Always include "log₂" if the data is log-transformed
- Use shorthand: "log₂(RNA/DNA)", "log₂ enrichment", "MRL"
- Include units when they exist: "pg/mL", "% of control", "ribosomes"

**In your methods section:**
- Spell out EXACTLY what was measured (RNA-seq? DNA-seq? Fluorescence?)
- Explain the normalization (control sequences? spike-ins? reference promoter?)
- Give the formula: log₂(X/Y) where X = ... and Y = ...

**In your results text:**
- You can drop "log₂" if it's clear from context: "Variant X showed 3.5-fold enrichment"
- For non-log values, be explicit: "MRL = 7.2 ribosomes" or "158% of control"

**Common mistakes to avoid:**
- ❌ Saying "fold-change" when you mean "log₂ fold-change" (3.5 vs. 2^3.5 = 11.3×)
- ❌ Writing "log" without specifying base (always use log₂ or log₁₀)
- ❌ Forgetting to mention normalization (what's your denominator?)
- ❌ Mixing linear and log scales in the same figure without labels

### Quick Reference Card

When you're writing your paper:

```
Measurement Type          Write in Figure          Write in Methods
─────────────────────────────────────────────────────────────────
Promoter strength         log₂(YFP/RFP)            "log₂ ratio of test to reference fluorescence"
Transcription             log₂(RNA/DNA)            "log₂ ratio of RNA-seq to DNA-seq reads"
Translation (selection)   log₂ enrichment          "log₂(post-selection / pre-selection counts)"
Translation (direct)      MRL (ribosomes)          "mean ribosome load from polysome profiling"
Relative activity         % of control             "(test/control) × 100%"
Absolute protein          pg/mL                    "protein concentration by ELISA"
mRNA stability            log₂ fold-change         "log₂(count / initial count)"
```

The golden rule: **Be consistent, be explicit, and always define your denominator.**

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