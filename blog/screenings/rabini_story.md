@def title = "Systematic Rules for Maternal mRNA Clearance in Zebrafish (bio version)"
@def published = "27 January 2026"
@def tags = ["rna", "screening"]

# Systematic Rules for Maternal mRNA Clearance in Zebrafish
## Technical Version (for biology specialists)

## The Biological Question
During the maternal-to-zygotic transition (MZT):
- Maternal mRNAs must be cleared on precise schedules
- Zygotic genome activates at ~3-4 hpf
- Each mRNA has a unique degradation profile

**The Question**: What 3'UTR sequence elements encode these dynamics?

---

## Two Temporal Degradation Programs

### Early-Onset Program (Maternal, 0-4 hpf)
- Active from fertilization
- Operates via maternally deposited factors
- **Cis-regulatory elements**:
  - Poly-U stretches → **Stabilizing**
  - UUAG motifs → **Stabilizing**
  - GC-rich sequences → **Destabilizing**

### Late-Onset Program (Zygotic, 4-10 hpf)
- Initiates after zygotic genome activation (ZGA)
- Requires zygotically transcribed factors
- **Cis-regulatory elements**:
  - miR-430 seed (GCACUU) → **Destabilizing**
  - AU-rich elements (AUUUAU) → **Destabilizing**
  - Pumilio recognition elements (UGUAHAUA) → **Destabilizing**

---

## A Comprehensive Sequence Grammar

**Highly systematic discovery**:

1. **Massively parallel reporter assay (MPRA)**: Tested ~35,000 3'UTR sequences (110nt each) simultaneously
   - Injected into 1-cell embryos
   - Measured mRNA levels hourly for 10 hours
   - Tracked both adenylated (A+) and deadenylated (A-) reporters

2. **Six dominant regulatory motifs** identified across all sequences

3. **Linear regression models** decode sequence → degradation dynamics:
   - Uses k-mer composition (3-7nt motifs)
   - Explains **70% of variance** in mRNA stability
   - Motifs act **additively**: more sites → stronger effect

4. **Validated by synthesis**: 
   - Designed artificial 3'UTRs with predicted half-lives
   - Experimental measurements matched predictions (r = 0.68)
   - Confirmed gain/loss-of-function effects of individual motifs

---

## Regression Model Framework

**Approach**: *k*-mer-based linear regression

$$\beta_g = w_0 + \sum_k N_{kg} \cdot w_k$$

Where:
- $\beta_g$ = degradation rate of reporter $g$
- $N_{kg}$ = count of *k*-mer $k$ in 3'UTR of reporter $g$
- $w_k$ = learned weight for *k*-mer $k$

**Two separate models**:
- **Early-onset model**: Trained on class I reporters (constant degradation)
  - 1,264 k-mers with non-zero weights
  - Explains 67% variance (A-), 38% variance (A+)
  
- **Late-onset model**: Trained on classes II & III (accelerated after ZGA)
  - 718 k-mers with non-zero weights  
  - Explains 29% variance (A-), 38% variance (A+)

**Combined performance**: 70% of total variance explained

**Key insight**: Short sequence elements (3-7nt) are sufficient to predict mRNA fate during MZT

---

## Summary for Presentation

**Core finding**:  
Six short sequence motifs in 3'UTRs govern mRNA clearance through two temporally distinct degradation programs (maternal and zygotic), creating predictable sequence-to-stability relationships.

**Timeline**:
```
hpf:      0────3────6────10
         │ Early │  Late  │
Program:  Maternal  Zygotic
Signals:  U/UUAG   miR-430
          GC-rich  ARE/PUM
```

**Why this matters**:
- First **systematic, quantitative** map of cis-elements controlling maternal mRNA clearance
- **Predictive models** enable rational design of mRNA stability
- Generalizable MPRA + regression framework ("UTR-Seq") for studying post-transcriptional regulation

**The power**: Not just correlation—causation confirmed by synthetic biology. You can now *engineer* mRNA half-lives by rational 3'UTR design.