@def title = "Units in Rabani et al. (2017) Data"
@def published = "19 January 2026"
@def tags = ["data-processing"]

# Units in Rabani et al. (2017) Data

## **UNITS: Log₂ normalized reporter mRNA count**

The numbers in your data table are **NOT raw counts**. They are:

**log₂(normalized reporter mRNA count / initial fitted reporter mRNA count)**

- **What's being measured**: Each specific reporter mRNA (e.g., S0_M_T11 is one reporter with a specific 3'UTR sequence)
- **Scale**: log₂ 
- **Reference point**: 0 = fitted initial count of that reporter (X₀)
- **Original measurement**: UMI counts (unique molecular identifiers from RNA-seq)
  - Each UMI ≈ one copy of that reporter mRNA molecule
- **Normalization**: Spike-in controls (known mRNA quantities: 50 fg down to 3.125 fg)
  - Converts UMI counts → estimated number of that reporter mRNA
- **Transformation**: log₂ applied after normalization

---

## What This Means Practically

**The values tell you**: How many copies of that specific reporter mRNA remain compared to the starting count, on a log₂ scale.

Each row (e.g., S0_M_T11) is tracking **one specific reporter sequence** - a GFP mRNA with a particular 3'UTR attached.

### Quick Reference
- **0** = At initial level
- **+1** = 2× initial level (twice as much)
- **-1** = 0.5× initial level (half remains)
- **-2** = 0.25× initial level (quarter remains)
- **-3** = 0.125× initial level (1/8 remains)
- **-5** = 0.03× initial level (~3% remains - heavily degraded)

### Converting to Fold-Change in Molecule Count
**Fold-change from initial = 2^(value)**

So a value of -3.0 means: 2^(-3) = 0.125 = 12.5% of original mRNA molecules remain.

---

## Examples from Your Data

**S0_M_T11** (late-onset degradation):
```
This specific reporter starts with ~2× its fitted baseline
1h: 0.98    → 2× starting count of S0_M_T11 reporter
4h: -0.23   → 0.85× (slight decay of this reporter)
6h: -1.52   → 0.35× (this reporter dropped to 35%)
10h: -5.33  → 0.025× (only 2.5% of this reporter left)
```

**S0_M_T100** (early-onset degradation):
```
This reporter starts high but degrades immediately
1h: 2.55    → 5.8× starting count of S0_M_T100 reporter
4h: -4.02   → 0.06× (this reporter down to 6%)
10h: -5.33  → 0.03× (3% of this reporter remains)
```

---

## The Processing Pipeline

1. **Raw data**: UMI counts from sequencing (each UMI ≈ 1 mRNA molecule)
2. **Spike-in normalization**: Calibrated to known mRNA molecule counts (converted from femtograms)
3. **Fitted initial count (X₀)**: Each reporter gets its own baseline molecule count
4. **Log₂ transformation**: log₂(normalized count / X₀)
5. **Result**: The numbers you see in the table

---

## Time Points

Column headers (1h, 2h, etc.) = **hours post fertilization (hpf)**

- 1-2 hpf: Early development
- 3-5 hpf: Zygotic genome activation (ZGA)
- 6-10 hpf: Post-ZGA

Many reporters crash between 4-6h when zygotic degradation machinery activates.