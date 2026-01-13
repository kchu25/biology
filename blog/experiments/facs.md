@def title = "FACS Explained: Computer Scientist Edition"
@def published = "12 January 2026"
@def tags = ["experimental-biology"]

# FACS Explained: Computer Scientist Edition

## Quick Overview

**FACS = Fluorescence-Activated Cell Sorting**. Think of it as a "smart filter" that physically separates cells based on how brightly they glow under specific wavelengths of light.

---

## General Concept

### The Basic Idea
Imagine you have millions of cells, and some express more GFP (Green Fluorescent Protein) than others. FACS lets you:

1. **Measure** how bright each cell glows (fluorescence intensity)
2. **Sort** cells into different collection tubes based on brightness
3. **Collect** specific populations for downstream analysis

### The Process

```
Cells in suspension → Laser excitation → Fluorescence detection → 
Charge droplets → Deflect by electric field → Collect in bins
```

**Key analogy**: It's like a coin sorter that separates pennies, nickels, and dimes—except instead of size/weight, it uses fluorescence as the sorting criterion.

---

## In This Paper's Context

### What They Did

The researchers had cells with different 5' UTRs driving GFP expression. They wanted to identify which 5' UTRs made cells produce **more** GFP protein.

**Their sorting strategy:**
- **Bin 1**: Top 2.5% (brightest cells)
- **Bin 2**: 2.5-5% (very bright)  
- **Bin 3**: 5-10% (bright)
- **Bin 4**: 0-100% (unsorted control)

### About That Notation

You spotted something interesting! Looking at the paper, they describe:
- "top 2.5%, 2.5–5%, 5–10%, and 0–100% (unsorted)"

**This is NOT a typo.** Here's what it means:

| Bin | Percentile Range | Meaning | Example (out of 1000 cells) |
|-----|------------------|---------|------------------------------|
| 1 | Top 2.5% | The absolute brightest 2.5% of cells | 25 brightest cells |
| 2 | 2.5-5% | The next tier (cells ranked 2.5-5% from the top) | Cells ranked 26-50 |
| 3 | 5-10% | Third tier (cells ranked 5-10% from the top) | Cells ranked 51-100 |
| 4 | 0-100% | Everything (unsorted control) | All 1000 cells |

### Visual Breakdown

Imagine ranking all cells by GFP brightness from brightest to dimmest:

```
[Bin 1: Top 2.5%]━━━━┓
[Bin 2: 2.5-5%]━━━━━┫ Collected separately
[Bin 3: 5-10%]━━━━━━┛
[...90% dimmer cells...]
└─ All cells together = Bin 4 (0-100%, unsorted)
```

**Key insight**: Bins 1-3 are **physically separated** during sorting. Each goes into its own collection tube. Bin 4 is just a sample taken **before sorting** to serve as the baseline.

### The Percentile Notation Explained

When they say "top 2.5%", they mean:
- Rank ALL cells by fluorescence intensity (bright → dim)
- Take the top 2.5% (if you have 100,000 cells, that's 2,500 cells)
- That's Bin 1

Then "2.5-5%" means:
- The cells ranked from position 2.5% to position 5%
- These are bright, but not AS bright as Bin 1
- If you have 100,000 cells, this is cells #2,501 through #5,000
- That's Bin 2

**Yes, these are mutually exclusive** (except for bin 4, which contains everything and serves as a reference).

The "0-100%" bin isn't really a sorted "bin"—it's the **pre-sort population** used as a baseline. They compare the enrichment of specific 5' UTR sequences in the top bins versus this unsorted control.

---

## Why This Design?

### Multiple Bins Strategy

They used 3 separate bins instead of just one "top 10%" bin because:

1. **Reduces false positives**: A 5' UTR must be enriched in ALL three top bins to be considered a hit
2. **Better resolution**: Can distinguish between "really good" and "pretty good" 5' UTRs
3. **Statistical power**: More data points for validation

### The Math Behind It

For each 5' UTR sequence, they calculated:

$$\text{Enrichment} = \log_2\left(\frac{\text{Count in top bin}}{\text{Count in unsorted}}\right)$$

A good 5' UTR should have high enrichment in **all three** top bins.

---

## Key Technical Points

### Why GFP?
- **Easy to measure**: Just shine blue light, detect green fluorescence
- **Non-toxic**: Doesn't kill cells
- **Quantitative**: Brightness ∝ amount of protein produced
- **High-throughput**: Can process millions of cells

### The Sorting Mechanism
1. Cells flow single-file through a tiny nozzle
2. Laser hits each cell and measures fluorescence
3. Computer decides which bin the cell belongs to
4. Droplet containing that cell gets electrically charged
5. Charged droplet deflects toward the correct collection tube

**Throughput**: ~10,000-50,000 cells/second

---

## Why This Matters for the Paper

### The Goal
Find 5' UTR sequences that boost protein expression.

### The Challenge
They had ~12,000 different 5' UTRs to test. Can't test them one-by-one!

### The Clever Solution: Pooled Screening

> **Wait, doesn't FACS sort individual cells? How do you test 12,000 variants at once?**
>
> Great question! You're right that FACS sorts one cell at a time—but here's the trick:
>
> **They DON'T need to test each 5' UTR separately.** Instead:
>
> 1. **Create a pooled library**: Mix all 12,000 different 5' UTR variants together
> 2. **Each cell gets ONE random variant**: When they transfect cells, each cell receives just one of the 12,000 5' UTR constructs (driving GFP)
> 3. **Sort by brightness**: FACS separates bright cells from dim cells (doesn't know or care which 5' UTR is inside)
> 4. **Sequence to identify winners**: After sorting, extract DNA from each bin and sequence it
>    - If you find 5' UTR #4523 is **super common** in the "bright" bin → that UTR is good!
>    - If 5' UTR #891 is **barely present** in the "bright" bin → that UTR is weak
>
> **Key insight**: You're not sorting 12,000 times. You're sorting **millions of cells** (each carrying one of 12,000 variants), then using DNA sequencing afterward to figure out which variants ended up in which bins.
>
> **Throughput numbers:**
> - FACS can process: ~10,000-50,000 cells/second
> - They need: >300,000 cells total (>25× coverage of 12,000 variants)
> - Time to sort: Just a few minutes!
> - Bottleneck: NOT the FACS, but the DNA sequencing afterward

### The Math

With 12,000 variants and 25× coverage:
- Need at least 300,000 cells sorted
- At 20,000 cells/second → takes ~15 seconds of sort time
- FACS is actually **perfect** for this throughput!

> **But wait—what about imbalanced representation?**
>
> Sharp question! This is a real concern. Here's the issue and solution:
>
> **The Problem:**
> - When you randomly distribute 12,000 variants across 300,000 cells (25× coverage)
> - Some variants might end up in 30 cells (good representation)
> - Others might end up in only 15 cells (undersampled)
> - If a variant is rare in your starting population, you might miss it in the final count
>
> **Their Solution:**
> They maintained **>25-fold coverage** throughout. This means:
> - Start with way more than 25 cells per variant (probably 50-100×)
> - Even after sorting only the top 10%, you still have enough cells per variant to count accurately
> - Statistical methods (like DESeq2, which they used) are designed to handle count variability
>
> **The key calculation:**
> - Top 10% bin from 300,000 cells = 30,000 cells collected
> - 30,000 cells ÷ 12,000 variants = ~2.5 cells per variant in the top bin
> - That's barely enough! So they likely started with many more cells (the paper says ">25-fold more cells than library size")
> - Probably started with 1-3 million cells → top 10% = 100,000-300,000 cells → ~8-25 cells per variant per bin
>
> **Why ratios help but don't fully solve it:**
> - Using enrichment ratios (Bin 1 count / Control count) normalizes for starting abundance
> - If variant X had 100 cells initially and 25 in top bin = 25% enrichment
> - If variant Y had 50 cells initially and 12 in top bin = 24% enrichment (similar!)
> - BUT if variant Z only had 5 cells initially and 2 in top bin = 40% (noise!)
>
> **Their safeguards:**
> 1. High coverage (>25×, likely much higher)
> 2. Biological replicates (two different cell lines)
> 3. Multiple bins (must be enriched in ALL three top bins)
> 4. Statistical testing (DESeq2 accounts for count variability)
> 5. Experimental validation afterward (only tested the top hits)

**The real power**: Combining FACS (physical separation) + Next-Gen Sequencing (identification) lets you test thousands of variants in a single experiment.

> **Do they use UMIs (Unique Molecular Identifiers)?**
>
> **No, they don't use UMIs in this paper.** Here's what they actually do and why:
>
> **Their sequencing approach:**
> 1. **Extract genomic DNA** from cells in each bin
> 2. **PCR amplify** the 5' UTR region (with barcodes to label which bin it came from)
> 3. **Sequence** the amplicons on Illumina NextSeq
> 4. **Count** how many times each 5' UTR sequence appears
>
> **Why no UMIs here:**
> - UMIs are typically used to distinguish **PCR duplicates from true biological duplicates**
> - In single-cell RNA-seq, UMIs tell you if you sequenced the same original RNA molecule twice (bad) vs two different RNA molecules (good)
> - **In this experiment:** Each cell has exactly ONE copy of the 5' UTR integrated into its genome
> - PCR bias is less critical because they're counting **cells** (not transcripts)
> - The question is: "How many cells with 5' UTR #4523 ended up in the bright bin?" not "How many transcripts were made?"
>
> **What mitigates PCR bias:**
> - They optimized PCR conditions for "unbiased amplification" (mentioned in methods)
> - They normalize counts across bins using DESeq2 (handles sequencing depth differences)
> - Each 5' UTR sequence itself acts as a natural "barcode"—you're just counting unique sequences
>
> **When you WOULD want UMIs in this type of experiment:**
> - If measuring RNA levels (not DNA) → UMIs distinguish molecules
> - If worried about severe PCR bias → UMIs let you collapse PCR duplicates
> - Some newer FACS screening methods do use UMIs for extra precision
>
> **The crucial insight: Ratios save you!**
> You're absolutely right—**using ratios is the key reason they don't need UMIs.**
>
> **Wait, do you measure before AND after PCR?**
>
> Ah, I see the confusion! **No, they don't measure pre-PCR vs post-PCR.** Here's what actually happens:
>
> **The actual workflow:**
> 1. Extract genomic DNA from each bin (bright bin, control bin, etc.)
> 2. PCR amplify ALL bins (same number of cycles, same conditions)
> 3. Sequence the PCR products
> 4. Count reads for each variant in each bin
> 5. **Compare ratios ACROSS bins** (not before/after PCR)
>
> **The ratio that matters:**
> - **Bright bin count / Control bin count** ← This is what they calculate
> - NOT: "after PCR / before PCR"
>
> **Why PCR bias cancels in CROSS-BIN ratios:**
> - Both bins go through the SAME PCR process
> - If variant X has 2× PCR bias:
>   - Control bin: starts with 100 DNA molecules → after PCR → 200 reads
>   - Bright bin: starts with 50 DNA molecules → after PCR → 100 reads  
>   - Ratio: 100/200 = 0.5 ✓ (correct! Because both had the same 2× bias)
>
> **What you might be thinking of:**
> - Some protocols DO measure DNA concentration before/after PCR to normalize
> - Or use "spike-in" controls to calibrate PCR efficiency
> - But this paper just relies on consistent PCR conditions across all bins
>
> **The key assumption:**
> PCR bias for any given sequence is **the same** whether you're amplifying DNA from the bright bin or control bin. Since all bins are processed identically (same primers, same cycles, same conditions), this holds true.
>
> **Bottom line:** They're taking ratios **between bins** (both after PCR), not measuring before/after PCR. The bias cancels because it affects each bin equally!

---

## Interpreting FACS Scatter Plots

> **What's the PURPOSE of these scatter plots?**
>
> **Short answer: Quality control and deciding where to draw the lines.**
>
> The scatter plots let you:
> 1. **Check your cells are healthy** (exclude dead cells, debris, clumps)
> 2. **Visualize the distribution** (see how bright/dim your cells are)
> 3. **Set your sorting gates** (decide where "top 2.5%" actually starts)
> 4. **Verify the sort worked** (confirm you actually separated populations)
>
> **Think of it like:** Before sorting mail by weight, you'd want to see a histogram of all package weights to decide "heavy packages = >10 lbs" and check you're not accidentally sorting rocks instead of packages.

> **What are those scatter plots showing?**
>
> FACS machines measure multiple properties of each cell simultaneously. The classic scatter plot shows:
>
> **The typical 2D plot you see:**
> - **X-axis**: Often "Forward Scatter" (FSC) = cell size
> - **Y-axis**: Often "Side Scatter" (SSC) = cell granularity/complexity
> - **Each dot** = one cell
>
> **What the axes mean:**
> - **Forward Scatter (FSC)**: Light that goes straight through → bigger cells scatter more light → larger FSC value
> - **Side Scatter (SSC)**: Light that bounces sideways → granular/complex cells (lots of internal structures) scatter more → higher SSC value
> - **Fluorescence channels**: Instead of FSC/SSC, you might see GFP intensity, RFP intensity, etc.
>
> **Common plot types:**
>
> 1. **FSC vs SSC (the "gating" plot)**
>    - Used to select healthy cells and exclude debris/dead cells
>    - Debris = low FSC + low SSC (small, simple)
>    - Healthy cells = cluster in the middle
>    - Doublets (two cells stuck together) = high FSC
>
> 2. **GFP vs Cell Count (histogram)**
>    - X-axis: GFP brightness
>    - Y-axis: Number of cells
>    - Shows the distribution of fluorescence
>    - You can draw "gates" to define bins: "top 2.5%" = rightmost peak
>
> 3. **GFP vs RFP (dual fluorescence)**
>    - Each axis is a different color channel
>    - Quadrants show: GFP+/RFP+, GFP+/RFP-, GFP-/RFP+, GFP-/RFP-
>    - Useful for co-expression analysis
>
> **Reading the scatter:**
> - **Tight cluster** = homogeneous population (all cells similar)
> - **Spread out** = heterogeneous (lots of variation)
> - **Multiple clusters** = distinct cell populations
> - **Diagonal line** = correlated properties (e.g., bigger cells express more protein)
>
> **In this paper's context:**
> - They'd show a histogram of GFP intensity
> - Draw vertical lines to mark the bins: top 2.5%, 2.5-5%, 5-10%
> - Cells to the right of each line go into that bin
>
> **Pro tip:** The "cloud" of dots tells you about biological variation. A tight cloud = consistent expression. A wide spread = noisy system or genuine cell-to-cell differences.

---

## Bottom Line

**FACS** = High-throughput cell sorter based on fluorescence  
**In this paper** = Tool to identify which 5' UTRs produce the most protein  
**The bins** = Mutually exclusive tiers of brightness (except the control)  
**No typo** = The 0-100% is the pre-sort reference population

Think of it as `GROUP BY` in SQL, but for living cells based on how bright they glow! 🔬✨