@def title = "SPLIRENT: The Biology, The Data, The Pipeline"
@def published = "12 February 2026"
@def tags = ["rna", "screening"]

# SPLIRENT: The Biology, The Data, The Pipeline

## The Big Picture Motivation

### What problem is this solving?

Your cells have a problem. When they read a gene from DNA, the initial copy (called **pre-mRNA**) contains both useful chunks (**exons**) and junk filler (**introns**). The cell has to **cut out the introns and stitch the exons together** — a process called **splicing**.

But here's the twist: sometimes the cell has **multiple possible places** where it *could* make that cut. This is called **alternative splicing**, and it's a *huge* deal — it's how ~20,000 human genes produce over 200,000 different proteins. Get the splicing wrong, and you get diseases like spinal muscular atrophy, cancers, and neurological disorders.

The specific type of alternative splicing studied here is **alternative 5' splice site (a5ss) selection** — where there are multiple possible "cutting" points on the left (5') end of an intron:

```
Normal splicing (one choice):

    Exon 1          Intron              Exon 2
  ──────────[GT...................AG]──────────
             ↑                    ↑
         5' splice            3' splice
         site (SD1)           site
         "cut here"           "cut here"


Alternative 5' splicing (multiple choices):

    Exon 1          Intron              Exon 2
  ──────────[GT...................AG]──────────
             ↑
         SD1 "cut here" (primary)
                ↑
            SD2 "or here" (alternative)
                     ↑
                 SD3 "or here" (alternative)
```

If the cell cuts at SD1, the resulting mRNA is longer. If it cuts at SD2 or SD3, the mRNA is shorter (more of the exon is lost). Which site gets used depends on the **surrounding DNA sequence**, and the question is:

> **Given a DNA sequence, can we predict which splice site the cell will choose — and does the answer change depending on the cell type?**

That's what SPLIRENT does.

---

## The Experimental Design

### The Reporter Construct (The "Test Tube")

The Rosenberg et al. (2015) experiment is an MPRA (Massively Parallel Reporter Assay) — but instead of measuring gene expression levels (like the promoter/UTR MPRAs in [your blog post](https://kchu25.github.io/biology/blog/general/mpra/)), it measures **splicing decisions**.

They built a **synthetic reporter gene** based on the fluorescent protein **Citrine** (a yellow-green GFP variant). The clever part: they split Citrine into two exons and stuck a synthetic intron in between, with **randomized DNA** in the region around the splice donor:

```
The Reporter Construct (from the GenBank file: alternative-5ss.gb)
═══════════════════════════════════════════════════════════════════

 Kozak   ┌──── Citrine Exon 1 (519 bp) ────┐  Synthetic Intron (303 bp)     Citrine Exon 2   BGH polyA
  ATG    │  ...GFP-like coding sequence...  │                                (200 bp)
─────────┤                                  ├──────────────────────────────┤            ├──────────
         │                                  │                              │            │
         └──────────────────────────────────┘                              └────────────┘
                                        ↓
                              This junction region is where
                              the splice sites live:

                  ...CACAACATCGAGG│TGCTT...NNN...GGTCGACCC│AGGTTCGTG...NNN...│GAGGTATTCT...
                                 ↑          ↑             ↑           ↑      ↑
                                SD1      Random         SD2        Random   SD3
                               (pos 0)  Region 1    (pos +44)   Region 2  (pos +79)
                                        (25 bp)                  (25 bp)
```

The `N`s in the GenBank file represent the **randomized nucleotides** — 25 bases of completely random DNA in two regions (Mut1 and Mut2 in the GenBank annotations). Every different combination of random bases is a unique sequence variant.

### What Are SD1, SD2, SD3?

These are three **splice donor** sites — positions where the splicing machinery could cut. They all contain the **GT** dinucleotide (the universal "cut here" signal recognized by the spliceosome):

| Site | Position (relative) | Sequence context | What happens if used |
|------|---------------------|-----------------|---------------------|
| **SD1** | 0 (primary) | `...CGAGG`**GT**`GCTT...` | Standard splice — intron fully removed |
| **SD2** | +44 | `...CCCAGG`**TT**`CGTG...` | 44 nt of intron left in mRNA (incomplete intron removal) |
| **SD3** | +79 | `...GAG`**GT**`ATTC...` | 79 nt of intron left in mRNA (incomplete intron removal) |

All three sites are **inside the intron**, downstream of the exon 1 boundary. SD1 sits right at the exon–intron junction (position 0), so using it removes the intron cleanly. SD2 and SD3 are 44 and 79 nt further into the intron, so splicing there leaves that extra chunk of intronic sequence in the mature mRNA.

The DNA sequence around these sites — especially the two 25-bp random regions — determines which site the cell prefers to use.

---

## The Measurement Pipeline

Here is the full pipeline from synthesized DNA to the final label you see in the CSV:

```
┌─────────────────────────────────┐
│  1. SYNTHESIZE LIBRARY          │
│                                 │
│  ~265,000 unique DNA sequences  │
│  (each with different random    │
│   regions around splice sites)  │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  2. CLONE INTO PLASMID          │
│                                 │
│  Each sequence goes into the    │
│  Citrine reporter construct     │
│  (see GenBank file above)       │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  3. TRANSFECT INTO CELLS        │
│                                 │
│  Put the plasmid library into   │
│  4 different cell lines:        │
│    • HEK293 (human kidney)      │
│    • HELA (cervical cancer)     │
│    • MCF7 (breast cancer)       │
│    • CHO (hamster ovary)        │
│                                 │
│  Same library → 4 cell types    │
│  = 4 independent measurements   │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  4. CELLS EXPRESS THE GENE      │
│                                 │
│  DNA → pre-mRNA → SPLICING →    │
│  → mature mRNA                  │
│                                 │
│  The cell's splicing machinery  │
│  decides WHERE to cut.          │
│  Different sequences → different│
│  splice site choices.           │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  5. RNA-seq                     │
│                                 │
│  Extract mRNA from cells.       │
│  Sequence it. Map reads back    │
│  to the reporter construct.     │
│                                 │
│  KEY: Where a read starts/ends  │
│  tells you WHERE the splice     │
│  happened (which SD was used).  │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  6. COUNT READS PER SITE        │
│                                 │
│  For each sequence variant,     │
│  count how many RNA-seq reads   │
│  support splicing at each       │
│  nucleotide position.           │
│                                 │
│  This gives a "cut profile":    │
│  a vector of counts across      │
│  all possible splice positions. │
│                                 │
│  → This is the raw count        │
│    matrix in the data files     │
│    (shape: 264,647 × 362)       │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  7. COMPUTE THE LABEL           │
│                                 │
│  For each sequence, for each    │
│  cell type:                     │
│                                 │
│         reads at SD1            │
│  PSI = ─────────────────────    │
│        reads at SD1 + SD2 +     │
│        SD3 + all other sites    │
│                                 │
│  This is "Percent Spliced In"   │
│  or "SD1 usage ratio"           │
│  Range: 0.0 to 1.0             │
│  (dimensionless ratio)          │
└─────────────────────────────────┘
```

---

## The Label: What Does It Mean?

The label in the CSV files is called **`sd1_usage`** (splice donor 1 usage ratio). It's computed as:

$$\text{sd1\_usage} = \frac{\text{RNA-seq reads supporting splicing at SD1}}{\text{total RNA-seq reads across all splice sites}}$$

This is conceptually similar to **PSI (Percent Spliced In)**, a standard metric in splicing biology.

### What "reads at SD1" means quantitatively

**"Reads at SD1"** = RNA-seq reads where the splice junction lands exactly at the SD1 position. 

When the spliceosome cuts at SD1, mature mRNA has a junction spanning the exon-intron boundary:
```
Mature mRNA (after splicing at SD1):
  ...Exon1_bases]Exon2_bases...
                ↑ junction point = SD1 was used
```

RNA-seq reads that span this junction indicate splicing happened at SD1.

**Concrete example:**
- Sequence has 10,000 total RNA-seq reads from all splicing events
- 8,500 reads have junctions at SD1 (primary site)
- 1,200 reads have junctions at SD2 (alternative)
- 600 reads have junctions at SD3 (alternative)
- **sd1_usage = 8,500 / 10,300 = 0.825** (82.5% use SD1)

The formula includes SD1 in **both numerator and denominator** — it's asking: "**What fraction of all splicing events used SD1?**"

| Value | Meaning |
|-------|---------|
| `1.0` | 100% of mRNAs used the primary splice site SD1 — no alternative splicing |
| `0.0` | 0% used SD1 — all mRNAs used alternative sites (SD2, SD3, or other) |
| `0.5` | Half and half — equal competition between primary and alternative sites |

### Units

**Dimensionless ratio** (0 to 1). It's NOT a log-scale value. It's a direct proportion — think of it as a percentage divided by 100.

### How it's computed in the code

From `splirent_data_5ss.py`:
```python
def iso_normalizer(t):
    """t is a vector: [count_at_pos_0, count_at_pos_1, ..., count_at_pos_99, distal_count]"""
    iso = 0.0
    if np.sum(t) > 0.0:
        iso = t[0] / np.sum(t)   # t[0] = count at SD1, sum(t) = total counts
    return iso
```

The extractor pulls counts from position 140 (SD1) through position 239 (100 nt downstream) plus the last column (distal splicing). Position 0 in the extracted vector `t` is the count at SD1. The ratio `t[0] / sum(t)` gives the SD1 usage.

### Why 4 labels per sequence?

The **same library** of ~265K sequences was tested in **4 different cell types**. Each cell type has its own splicing machinery with slightly different preferences, so the same sequence can give different SD1 usage ratios across cell types:

```
Same DNA sequence: GGGCATCGACTTCAAGGAGG...

  HEK cells:  sd1_usage = 0.85  (strongly prefers SD1)
  HELA cells: sd1_usage = 0.72  (mostly SD1, some alternatives)
  MCF7 cells: sd1_usage = 0.45  (roughly 50/50)
  CHO cells:  sd1_usage = 0.91  (very strong SD1 preference)
```

This is biologically interesting — it means splicing is not *just* determined by DNA sequence, but also by what proteins are present in each cell type.

### Label statistics (main dataset)

| Cell type | Mean sd1_usage | Std | Min | Max |
|-----------|---------------|-----|-----|-----|
| HEK | 0.224 | 0.308 | 0.0 | 1.0 |
| HELA | 0.244 | 0.328 | 0.0 | 1.0 |
| MCF7 | 0.184 | 0.287 | 0.0 | 1.0 |
| CHO | 0.265 | 0.337 | 0.0 | 1.0 |

Note that the mean is low (~0.2) — meaning for most random sequences, the primary splice site is NOT the dominant one. This makes sense: the randomized regions disrupt the natural signals that would normally direct splicing to SD1.

---

## The Sequence: What's In It?

Each sequence in the main dataset (`dataset_main.csv`) is **361 bp** and has this structure:

```
Position:  0                                  140              241                    361
           |────── Fixed upstream (140 bp) ─────|── Variable (101 bp) ──|── Fixed downstream (120 bp) ──|

           ├─────────── Citrine exon 1 ─────────┤                      ├─── Intron continuation ────────┤
           │                                    │                      │                                │
           GGGCATCGACTTCAAGGAGGAC...CGCCACAACATCGAGG│TGCTT...SD1...random1...SD2...random2...SD3...│ACAGAGTTTCCTTATTTG...
                                                    ↑                                               ↑
                                                  SD1 (pos 140)                                 SD3 region
```

### Breakdown

| Region | Position | Length | Content |
|--------|----------|-------|---------|
| Upstream background | 0–139 | 140 bp | End of Citrine exon 1 (constant for all sequences) |
| **Variable region** | **140–240** | **101 bp** | **Contains SD1, random region 1, SD2, random region 2, SD3** |
| Downstream background | 241–360 | 120 bp | Intron continuation (constant for all sequences) |

Within the variable region:

| Sub-region | Relative position | Length | Content |
|-----------|-------------------|--------|---------|
| SD1 splice donor | 140 | ~7 bp | Fixed GT + context |
| Random region 1 | 147–171 | 25 bp | **Randomized** (4^25 ≈ 10^15 possible sequences) |
| Fixed spacer | 172–189 | 18 bp | `GGTCGACCCAGGTTCGTG` |
| Random region 2 | 190–214 | 25 bp | **Randomized** |
| SD3 region + context | 215–240 | ~26 bp | Fixed, includes SD3 |

---

## The Aligned Datasets: What Are They?

The "aligned" datasets are a **reformulation** of the same data for a different ML task.

Instead of predicting "which fraction of mRNAs use SD1?", the aligned formulation asks: **"At this specific nucleotide position, is there splicing happening or not?"**

### How it works

For each of the 264,647 original sequences, the pipeline:

1. **Identifies every position with observed splice cuts** (positions where RNA-seq reads indicate splicing happened)
2. **Extracts a 260-bp window centered on that position**
3. **Records whether the position is:**
   - `fixed_0`: the primary splice donor SD1
   - `fixed_44`: the alternative splice donor SD2
   - `fixed_79`: the alternative splice donor SD3
   - `denovo_pos_X`: a position where splicing *was* observed (positive example)
   - `denovo_neg_X`: a position where splicing was *not* observed (negative example, randomly sampled)

```
Original sequence (361 bp):
─────────────────────[SD1]──────────[SD2]──────────[SD3]──────────────────────
                       ↑                ↑               ↑        ↑
                    fixed_0          denovo_pos_18   fixed_44  denovo_neg_86

For each ↑, extract a 260-bp window centered on it → one row in the aligned dataset
```

This produces many more rows (918K–2.7M depending on negative sampling) but shorter sequences (260 bp). The label is still computed the same way — splice usage ratio at the center position.

### Which aligned dataset?

| Dataset | Negative sampling ratio | Rows |
|---------|------------------------|------|
| `dataset_aligned.csv` | 0 (positives only) | 918,832 |
| `dataset_aligned_neg_rate_1.csv` | 1× | 1,837,664 |
| `dataset_aligned_neg_rate_2.csv` | 2× | 2,756,496 |

---

## How This Compares to Other MPRAs

If you've read about expression-measuring MPRAs (like in [your blog post](https://kchu25.github.io/biology/blog/general/mpra/)), here's a comparison:

| Aspect | Expression MPRA (e.g., de Boer) | Splicing MPRA (this dataset) |
|--------|-------------------------------|------------------------------|
| **What varies** | Promoter or UTR sequence | Intron/exon junction sequence |
| **What's measured** | How much protein is made | Where the mRNA is cut (spliced) |
| **Raw measurement** | Fluorescence or RNA/DNA read ratio | RNA-seq read positions (splice junctions) |
| **Label type** | log₂(expression ratio) | Splice site usage ratio (0–1) |
| **Label scale** | Log scale (spans orders of magnitude) | **Linear scale** (proportion) |
| **Normalization** | RNA/DNA or test/reference fluorescence | Count at site / total counts |
| **Task type** | Regression (predict expression level) | Regression (predict splicing ratio) |
| **Library size** | 10K–100M sequences | ~265K sequences |
| **Cell types** | Usually 1 | **4** (HEK, HELA, MCF7, CHO) |

The key difference: expression MPRAs ask "how *much* output?", while this splicing MPRA asks "which *form* of output?". Both are sequence → continuous value regression tasks.

---

## TL;DR Pipeline Diagram

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Synthesize  │     │   Clone into │     │  Transfect   │     │  Cells do    │
│  ~265K DNA   │────▶│   Citrine    │────▶│  into 4 cell │────▶│  SPLICING    │
│  sequences   │     │   reporter   │     │  types       │     │  (SD1/2/3)   │
│  (random     │     │   plasmid    │     │              │     │              │
│   regions)   │     │              │     │  HEK, HELA,  │     │  Machinery   │
│              │     │              │     │  MCF7, CHO   │     │  picks a     │
│              │     │              │     │              │     │  splice site │
└──────────────┘     └──────────────┘     └──────────────┘     └──────┬───────┘
                                                                      │
                     ┌──────────────┐     ┌──────────────┐           │
                     │  YOUR CSV    │     │  RNA-seq:    │           │
                     │              │     │  count reads │◀──────────┘
                     │  sequence,   │◀────│  at each     │
                     │  sd1_usage   │     │  nucleotide  │
                     │  (0.0–1.0)  │     │  position    │
                     │              │     │              │
                     │  264,647 rows│     │  Which reads │
                     │  × 4 cell    │     │  end at SD1? │
                     │    types     │     │  SD2? SD3?   │
                     └──────────────┘     └──────────────┘

                              sd1_usage = reads_at_SD1 / total_reads
```

---

## For the ML Person: Input/Output Summary

| | Description |
|---|---|
| **X (input)** | DNA sequence string, 361 characters from alphabet {A, C, G, T} |
| **Y (output)** | 4 floats in [0, 1]: sd1_usage for HEK, HELA, MCF7, CHO |
| **Task** | Multi-output regression |
| **Dataset size** | 264,647 samples |
| **Model** | CNN (convolutional neural network) on one-hot encoded sequence |
| **Loss** | KL divergence (on the splice cut distribution) |
| **Train/val/test** | Last 20K held out (10K val + 10K test), rest is train |
| **Paper** | Rosenberg et al., Cell 2015 (original HEK data); replicated in HELA, MCF7, CHO |

---

*Source: [Rosenberg et al., Cell 2015](https://doi.org/10.1016/j.cell.2015.09.054) · [SPLIRENT repo](https://github.com/johli/splirent)*
