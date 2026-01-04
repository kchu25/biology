@def title = "UTR-Seq Paper Overview (Rabani et al., 2017)"
@def published = "3 January 2026"
@def tags = ["rna", "binding-sites", "utr"]

# UTR-Seq Paper Overview (Rabani et al., 2017)

**Reference**: Rabani, M., Pieper, L., Chew, G.L., & Schier, A.F. (2017). Massively parallel reporter assay of 3′UTR sequences identifies in vivo rules for mRNA degradation. *Molecular Cell*, 68(6), 1083-1094.e5.

Hey! So this paper introduces a really cool technique called **UTR-Seq** that figures out how the sequences in mRNA molecules affect how quickly they get degraded in cells.

## The Big Picture

The researchers wanted to understand: **What parts of an mRNA's sequence tell the cell "hey, break me down now" versus "keep me around longer"?**

They focused on the **3′UTR** (the tail end of mRNA molecules) and studied this during early zebrafish embryo development—specifically when maternal mRNAs are being cleared out.

## What They're Trying to Predict

The paper is trying to predict **mRNA degradation rates**—basically, how fast different mRNAs get broken down over time. They measure two key things:

1. **Degradation rate (β)** - How quickly the mRNA disappears (they report this as half-life in hours)
2. **Onset time (t₀)** - When the degradation accelerates (some mRNAs stay stable early, then suddenly decay faster after ~3-5 hours)

## About Those Labels (1h, 2h, etc.)

Yes! Each sequence in their dataset has associated measurements at different time points—**1 hour post-fertilization (hpf), 2hpf, 3hpf**, and so on up to 10hpf. These time-series measurements let them track how each mRNA decays over time, which is way more informative than just a single snapshot.

**hpf = hours post-fertilization** - it's the standard way developmental biologists measure time in early embryos. It starts counting from when the egg is fertilized:
- **1hpf**: 1 hour after fertilization
- **2hpf**: 2 hours after fertilization
- **3-5hpf**: When zebrafish genome activation happens (zygotic transcription kicks in)
- **10hpf**: Late in early development

They're tracking how the injected mRNA reporters decay during the first 10 hours of zebrafish embryo development.

### What Are They Actually Measuring?

At each time point, they measure **mRNA abundance** - how much of each reporter mRNA is still present in the embryo.

**The measurement process:**
1. Inject synthetic mRNA reporters into zebrafish embryos
2. Collect embryos at different time points (1h, 2h, 3h, etc.)
3. Extract total RNA and sequence the 3′UTR regions
4. Count reads using **UMIs (Unique Molecular Identifiers)** to get absolute molecule counts
5. Normalize using spike-in controls (known quantities of control mRNAs)

> **Why "Unique Molecular Identifiers"?**
> 
> UMIs are random DNA barcodes (like 8 random nucleotides: ACGTTAGC) that get attached to each individual mRNA molecule during the reverse transcription step. Here's why they're crucial:
> 
> **The problem**: When you do PCR amplification before sequencing, some molecules get copied way more than others (PCR bias). If you just count sequencing reads, a single mRNA molecule that got lucky and amplified 100x looks like 100 different molecules!
> 
> **The solution**: Tag each original molecule with a random barcode *before* any amplification. Now after PCR, you can look at the barcodes - if you see the same barcode 100 times, you know it came from just 1 original molecule. Different barcodes = different original molecules.
> 
> So "Unique Molecular Identifier" literally means: each original molecule gets a unique ID tag that lets you identify and count it, even after all the amplification noise. It's the difference between counting *sequencing reads* (inflated by PCR) vs counting *actual original molecules* (what you really care about).

**Units:** The values are **normalized UMI counts** - essentially representing the number of mRNA molecules of each reporter present in the sample. After normalization to spike-ins, these are on a common scale across all time points, allowing direct comparison of how levels change over time.

Think of it like this: if you start with 1000 molecules at 1hpf and measure 500 at 2hpf, the mRNA is decaying with a half-life of ~1 hour.

### Data Format

Here's what their data looks like:

**Raw MPRA Data Structure:**

| Reporter ID | 3′UTR Sequence (110nt) | 1hpf | 2hpf | 3hpf | 4hpf | 5hpf | 6hpf | 7hpf | 8hpf | 10hpf |
|------------|------------------------|------|------|------|------|------|------|------|------|-------|
| Reporter_1 | AUGC...UAAA | 1245 | 1189 | 1098 | 987 | 856 | 723 | 612 | 498 | 301 |
| Reporter_2 | GCUA...CUUG | 2341 | 2298 | 2156 | 1876 | 1234 | 789 | 445 | 289 | 134 |
| Reporter_3 | UUUU...GUAU | 891 | 883 | 879 | 871 | 865 | 854 | 847 | 839 | 821 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

*Note: Values are **normalized UMI counts** (proportional to actual mRNA molecule counts)*

**After Fitting Degradation Models:**

| Reporter ID | 3′UTR Sequence | Model Type | $X_0$ | $\beta$ (h⁻¹) | $t_0$ (hpf) | $\beta_0$ (h⁻¹) | Half-life (h) | Class |
|------------|----------------|------------|-------|---------------|-------------|------------------|---------------|-------|
| Reporter_1 | AUGC...UAAA | Early-onset | 1245 | 0.38 | - | - | 1.8 | I |
| Reporter_2 | GCUA...CUUG | Late-onset | 2341 | 0.63 | 4.2 | 0.08 | 1.1 | II |
| Reporter_3 | UUUU...GUAU | Early-onset | 891 | 0.05 | - | - | 13.9 | I |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

**For Regression Input (k-mer Features):**

| Reporter ID | AAA | AAC | AAG | AAU | ... | UUUUUUU | GCACUU | UAUUUAU | ... | $\beta$ (target) |
|------------|-----|-----|-----|-----|-----|---------|--------|---------|-----|------------------|
| Reporter_1 | 3 | 2 | 5 | 1 | ... | 0 | 1 | 0 | ... | 0.38 |
| Reporter_2 | 1 | 4 | 2 | 3 | ... | 0 | 2 | 1 | ... | 0.63 |
| Reporter_3 | 2 | 1 | 3 | 2 | ... | 3 | 0 | 0 | ... | 0.05 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

*21,824 possible k-mer features (all 3-7nt sequences)*

### Scale

- **90,000** designed sequences
- **~35,000** reporters with sufficient coverage
- **~31,000** successfully fitted with degradation models
- **9-10** time points per reporter
- **Two libraries**: A+ (with poly(A) tail) and A- (without)

## The Key Findings

They discovered **six major sequence signals** that control mRNA stability:

**Stabilizing signals (keep mRNA around longer):**
- Poly-U sequences (lots of U's)
- UUAG sequences

**Destabilizing signals (make mRNA decay faster):**
- GC-rich sequences (early degradation)
- miR-430 binding sites (late degradation, after genome activation)
- AU-rich elements (AREs - late degradation)
- Pumilio recognition sites (late degradation)

## The Prediction Models - How They Work

Yes, they built prediction models, but it's more sophisticated than just regressing on individual time points! Here's the breakdown:

### Step 1: Model the Degradation Kinetics First

They **don't** regress on 1h, 2h individually. Instead, they:

1. **Fit degradation models** to the time-series data (all time points together) for each mRNA
2. Extract **kinetic parameters** from these models:
   - $\beta$ (degradation rate)
   - $t_0$ (onset time, if applicable)
   - $X_0$ (initial level)

They use first-order exponential decay kinetics:

$\frac{dX}{dt} = -b(t) \cdot X(t)$

Two models are tested:
- **Early-onset model**: Constant rate throughout ($b(t) = \beta$)
- **Late-onset model**: Step function that changes at onset time

$b(t) = \begin{cases} \beta_0 & t \leq t_0 \\ \beta & t > t_0 \end{cases}$

They pick the best model for each mRNA using likelihood ratio tests.

### Step 2: Regression on Kinetic Parameters

Now here's the clever part—they regress **from sequence to degradation rates**, not directly to time points!

**Input**: k-mer composition (counts of all 3-7 nucleotide sequences in the 3′UTR)

**Output**: Degradation rate ($\beta$)

The regression equation:

$\beta_g = w_0 + \sum_{k} N_{kg} \cdot w_k$

Where:
- $N_{kg}$ = number of times k-mer $k$ appears in sequence $g$
- $w_k$ = weight for each k-mer (learned by the model)

### Step 3: LASSO Regularization

They use **lasso-regularized linear regression** (not just plain regression!) which:
- Automatically selects important k-mers (feature selection)
- Shrinks most k-mer weights to exactly zero
- Keeps only the most informative sequence signals

Their two-step approach:
1. **First pass**: LASSO on high-confidence reporters (up to 20,000) to identify relevant k-mers
2. **Second pass**: Standard linear regression on ALL reporters, but only using the k-mers selected in step 1

**No neural nets or deep learning here!** They kept it simple and interpretable:
- Linear regression with LASSO regularization
- Each k-mer contributes independently and additively (no interactions modeled)
- You can directly see which sequences matter and by how much (the weights $w_k$)
- Published in 2017, before the deep learning craze fully took over biology

This simplicity is actually a feature, not a bug—it makes the model interpretable and biologically meaningful. You can look at the weights and say "poly-U sequences stabilize mRNAs by X amount" rather than having some black-box neural net.

### Two Separate Models

They actually built **two different regression models**:

1. **Early-onset regression**: Trained on class I reporters (constant degradation)
2. **Late-onset regression**: Trained on all reporters, predicts the accelerated rate after genome activation

### Performance

- Early-onset model explains **60%** of variance ($r^2 = 0.60$)
- Late-onset model explains **43%** of variance ($r^2 = 0.43$)
- Combined predictions explain **70%** of overall variation

They validated using 10-fold cross-validation—training on 90% of data, testing on the remaining 10%, repeated multiple times.

### The Cool Part

The models work! They can predict degradation rates just from sequence, and they even used these predictions to **design artificial 3′UTRs** with specific decay patterns—stable, unstable, or transitional—and experimentally validated them.

## Two Temporal Programs

They found two distinct degradation programs:
- **Early-onset**: Constant degradation from the start (maternally encoded)
- **Late-onset**: Stable initially, then rapid degradation after ~3-5 hours when the embryo's own genome kicks in

Pretty neat system for understanding how cells control which mRNAs stick around and which get destroyed!