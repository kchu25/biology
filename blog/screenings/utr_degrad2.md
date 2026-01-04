@def title = "Understanding A+ vs A- Reporters in the UTR-Seq Study"
@def published = "4 January 2026"
@def tags = ["rna", "screening", "utr"]

# Understanding A+ vs A- Reporters in the UTR-Seq Study

**Paper:** Massively parallel reporter assay of 3′UTR sequences identifies in vivo rules for mRNA degradation

**Reference:** Rabani et al., *Molecular Cell* (2017), PMC5994907

---

So you've got two supplementary files - one labeled A+, one labeled A-. Here's what they represent:

## The Quick Answer

**A+ = mRNA reporters WITH poly(A) tails** (36 nucleotides long)

**A- = mRNA reporters WITHOUT poly(A) tails**

## Why Two Versions?

The researchers wanted to understand how poly(A) tails affect mRNA stability. Think of the poly(A) tail as a protective cap on the end of mRNA - it matters for how long the mRNA sticks around in the cell.

They took the same library of ~90,000 different 3′UTR sequences and made two versions:
- One with poly(A) tails attached (A+)
- One without them (A-)

Then they injected both into zebrafish embryos and tracked how quickly each version degraded over time.

## What They Found

**A+ mRNAs lasted longer** - about 3.8 hours on average vs 2.4 hours for A- mRNAs.

But here's the interesting part: different 3′UTR sequences responded differently to having a poly(A) tail. Some needed it for stability, others didn't care much. This helped them classify 3′UTRs into three groups based on how they interact with poly(A) tails.

## So Are They Replicates?

**No** - A+ and A- are two different experimental conditions being compared, not replicates of the same thing.

However, *within* each file, there are some replicate measurements at various timepoints (5 replicates for A+, 2 for A-).

## About the Data Values

You might notice the values in the supplementary files aren't integers - they look like: 2.519, -1.8448, -0.43334, etc.

These are **normalized, log2-transformed UMI counts**, not raw counts. Here's the process:

1. **Raw UMI counts** - They counted unique molecular identifiers for each reporter
2. **Spike-in normalization** - They added 5 control mRNAs at known amounts (50fg down to 3.125fg in 2-fold steps) to each sample
3. **Calibration** - They fitted a regression model to the spike-ins to convert UMI counts to "expected mRNA molecules"
4. **Log transformation** - Values are on a log2 scale (which is why some are negative - these represent mRNAs with abundance <1 on the original scale)

From the methods: *"UMI counts were adjusted to represent the expected number of mRNA molecules in the sample... We fitted a generalized binomial linear regression model to counts of five control mRNAs that were added to samples at known quantities... and normalized by internal mRNA spike-ins."*