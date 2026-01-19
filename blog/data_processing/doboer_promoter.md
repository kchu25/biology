@def title = "de Boer et al. (2019) Yeast Promoter Expression Units Explained"
@def published = "19 January 2026"
@def tags = ["data-processing"]

# de Boer et al. (2019) Yeast Promoter Expression Units Explained

## YES - That's Exactly the Premise!

The **random test promoter** can have wildly different strength than the **reference promoter**, and that's the whole point of the measurement.

## The Mathematical Framework

### What's Actually Being Measured

For a single cell containing the plasmid:

$$
\text{YFP}_{\text{intensity}} = k_Y \cdot \alpha_{\text{test}} \cdot n \cdot \eta
$$

$$
\text{RFP}_{\text{intensity}} = k_R \cdot \alpha_{\text{TEF2}} \cdot n \cdot \eta
$$

Where:
- $\alpha_{\text{test}}$ = transcriptional activity of the test promoter (what we want to measure)
- $\alpha_{\text{TEF2}}$ = transcriptional activity of TEF2 reference promoter (constant)
- $n$ = plasmid copy number in this cell
- $\eta$ = cellular machinery efficiency (ribosomes, RNA polymerase, etc.)
- $k_Y, k_R$ = detection efficiency constants (fluorophore brightness, detector sensitivity)

### The Ratio Cancels Confounding Variables

$$
\frac{\text{YFP}_{\text{intensity}}}{\text{RFP}_{\text{intensity}}} = \frac{k_Y \cdot \alpha_{\text{test}} \cdot n \cdot \eta}{k_R \cdot \alpha_{\text{TEF2}} \cdot n \cdot \eta}
$$

$$
= \frac{k_Y}{k_R} \cdot \frac{\alpha_{\text{test}}}{\alpha_{\text{TEF2}}}
$$

Notice that $n$ and $\eta$ **cancel out completely**.

### Log Transform for Better Dynamic Range

$$
\text{Expression} = \log_2\left(\frac{\text{YFP}}{\text{RFP}}\right)
$$

$$
= \log_2(k_Y) - \log_2(k_R) + \log_2(\alpha_{\text{test}}) - \log_2(\alpha_{\text{TEF2}})
$$

$$
= C + \log_2(\alpha_{\text{test}})
$$

where $C = \log_2(k_Y/k_R) - \log_2(\alpha_{\text{TEF2}})$ is a constant offset.

## The Key Premise: $\alpha_{\text{test}}$ Varies Widely

For random promoter sequences:

$$
\alpha_{\text{test}} \in [0.01 \cdot \alpha_{\text{TEF2}}, 10 \cdot \alpha_{\text{TEF2}}]
$$

**This means:**
- Weak promoters: $\text{YFP} \ll \text{RFP}$ → Expression $\approx -7$ to $0$
- Medium promoters: $\text{YFP} \approx \text{RFP}$ → Expression $\approx 0$
- Strong promoters: $\text{YFP} \gg \text{RFP}$ → Expression $\approx 0$ to $+4$

The paper reports a ~50-fold range, which is:
$$
\log_2(50) \approx 5.64 \text{ units}
$$

## Why the Ratio Matters (Concrete Example)

### Scenario: Two cells with different copy numbers

**Cell 1:** High copy number
$$
n_1 = 50 \text{ plasmids}
$$

**Cell 2:** Low copy number
$$
n_2 = 10 \text{ plasmids}
$$

### Without normalization (measuring YFP only):

**Cell 1:**
$$
\text{YFP}_1 = k_Y \cdot \alpha_{\text{test}} \cdot 50 \cdot \eta = 50 k_Y \alpha_{\text{test}} \eta
$$

**Cell 2:**
$$
\text{YFP}_2 = k_Y \cdot \alpha_{\text{test}} \cdot 10 \cdot \eta = 10 k_Y \alpha_{\text{test}} \eta
$$

**Problem:** $\text{YFP}_1 = 5 \times \text{YFP}_2$ even though they have the **same promoter**!

### With normalization (measuring YFP/RFP):

**Cell 1:**
$$
\frac{\text{YFP}_1}{\text{RFP}_1} = \frac{k_Y \cdot \alpha_{\text{test}} \cdot 50 \cdot \eta}{k_R \cdot \alpha_{\text{TEF2}} \cdot 50 \cdot \eta} = \frac{k_Y \alpha_{\text{test}}}{k_R \alpha_{\text{TEF2}}}
$$

**Cell 2:**
$$
\frac{\text{YFP}_2}{\text{RFP}_2} = \frac{k_Y \cdot \alpha_{\text{test}} \cdot 10 \cdot \eta}{k_R \cdot \alpha_{\text{TEF2}} \cdot 10 \cdot \eta} = \frac{k_Y \alpha_{\text{test}}}{k_R \alpha_{\text{TEF2}}}
$$

**Solution:** Both cells give the **same ratio** despite 5-fold difference in copy number!

## The Actual Units

### Raw measurements (arbitrary units):

$$
\text{YFP} \in [10^2, 10^6] \text{ a.u.}
$$
$$
\text{RFP} \in [10^2, 10^6] \text{ a.u.}
$$

These are **digitized photon counts** from the photomultiplier tube.

### Final expression value:

$$
\text{Expression} = \log_2\left(\frac{\text{YFP (a.u.)}}{\text{RFP (a.u.)}}\right)
$$

**Unit:** Dimensionless (log of a dimensionless ratio)

**Range in their data:** Approximately $-3$ to $+15$

## Variance Decomposition

The total variance in YFP intensity can be decomposed as:

$$
\text{Var}(\text{YFP}) = \text{Var}(\alpha_{\text{test}}) + \text{Var}(n) + \text{Var}(\eta) + \text{Var}(\text{noise})
$$

By taking the ratio:

$$
\text{Var}\left(\frac{\text{YFP}}{\text{RFP}}\right) \approx \text{Var}(\alpha_{\text{test}}) + \text{Var}(\text{noise})
$$

The variance from $n$ and $\eta$ are **eliminated** because they affect both YFP and RFP equally.

## What They're Actually Testing

**Null hypothesis:** Random DNA sequences have no promoter activity
$$
H_0: \alpha_{\text{test}} = 0
$$

**Alternative:** Random sequences can have varying promoter strengths
$$
H_A: \alpha_{\text{test}} \sim \text{some distribution}
$$

They found that random sequences DO have activity, spanning:
$$
\alpha_{\text{test}} \in [0.01 \alpha_{\text{TEF2}}, 10 \alpha_{\text{TEF2}}]
$$

This ~1000-fold range ($\log_2(1000) \approx 10$) shows random DNA can accidentally create functional promoters.

## Summary

**The premise:** $\alpha_{\text{test}}$ can deviate significantly from $\alpha_{\text{TEF2}}$

**The solution:** Measure both simultaneously and take the ratio

**The math:** Confounding variables cancel, leaving only the signal of interest

**The units:** $\log_2(\text{dimensionless ratio})$ where the ratio is in arbitrary fluorescence units that cancel out

## Flow Cytometry Throughput

**YES - Flow cytometry can absolutely handle millions of cells!**

**Typical speeds:**
- Standard flow cytometers: **10,000 - 50,000 cells/second**
- High-throughput systems: **100,000 - 400,000 cells/second**
- State-of-the-art imaging flow cytometry: **>1,000,000 cells/second**

**In their experiment:**
- Analyzing 100 million promoters across multiple bins
- At 10,000 cells/second → 10 million cells in ~17 minutes
- At 50,000 cells/second → 10 million cells in ~3 minutes

They likely ran multiple sorts and pooled cells into bins based on YFP/RFP ratio, then sequenced the DNA from each bin to determine which promoter sequences gave which expression levels.

**The workflow:**
1. Millions of yeast cells flow through single-file
2. Each cell measured for YFP and RFP (takes microseconds)
3. Cells sorted into 18 expression bins in real-time
4. DNA extracted from each bin and sequenced
5. Map: promoter sequence → expression bin → log₂(YFP/RFP) value