@def title = "Cas13d Target RNA Binding: Proximal vs Distal Regions"
@def published = "7 November 2025"
@def tags = ["cas13d", "rna", "mutegenisis"]

# Cas13d Target RNA Binding: Proximal vs Distal Regions

## Data Structure Overview

**Note:** Analyzing your actual CSV data reveals variable total lengths (not all 75nt).

**Constant Prefix (30nt):**
```
GGGAATGGATCCACATCTACGAATTCGTAC
Position: 1-30
```

**Variable Region Structure:**
```
[5'-PFS: 3nt] + [Target: ~22nt] + [3'-PFS: variable]
```

**Constant Suffix (21nt):**
```
CGAGATCGGAAGAGCACACGT
Position: Variable (last 21nt of each sequence)
```

**Full Structure:**
```
[30nt Prefix] + [3nt 5'-PFS] + [~22nt Target] + [Variable 3'-PFS] + [21nt Suffix]
Total length varies: 73-77nt
```

### Example Breakdown (Row 5):
```
GGGAATGGATCCACATCTACGAATTCGTAC GCATAGAGAGGTTATCCGCTTC ACCGAGATCGGAAGAGCACACGT
[--------30nt Prefix----------] [-----22nt Target-------] [------25nt Suffix--------]
                                 |3nt|----19nt----|                  
                                 PFS    Target core
```

---

## Region Definitions

| Region | Position in 22nt Target | Position in Full Sequence | Length |
|--------|------------------------|---------------------------|---------|
| **Proximal** | 1-11 | 31-41 | 11 nt |
| **Distal** | 12-22 | 42-52 | 11 nt |

### Example (Row 1):
```
Full sequence:
GGGAATGGATCCACATCTACGAATTCGTACGCATAGAGAGGTTATCCGCTTCACCGAGATCGGAAGAGCACACGT

Structure:
[-----Prefix (1-30)-----][--Proximal (31-41)--][--Distal (42-52)--][--Suffix (53-75)--]
GGGAATGGATCCACATCTACGAATTCGTAC GCATAGAGAGG            TTATCCGCTTC         ACCGAGATCGGAAGAGCACACGT
```

---

## Key Findings: Effects on Cas13d Activity

### 1. **Distal Region (12-22) - Recognition "Seed" Region**

| Factor | Effect on Binding | Effect on Cleavage |
|--------|------------------|-------------------|
| Mismatches | **Strong NEGATIVE ⬇️⬇️** | NEGATIVE ⬇️ |
| RNA secondary structure | **Strong NEGATIVE ⬇️⬇️** | NEGATIVE ⬇️ |

**Mechanism:**
- Cas13d **initiates target recognition** at the distal end first
- Acts as internal "seed" region (analogous to PAM-adjacent seed in Cas9)
- Must remain **partially unstructured** for efficient binding
- More intramolecular base pairing → Lower binding affinity

### 2. **Proximal Region (1-11) - Nuclease Activation Region**

| Factor | Effect on Binding | Effect on Cleavage |
|--------|------------------|-------------------|
| Mismatches | **No effect ➡️** | NEGATIVE ⬇️ |
| RNA secondary structure | **No effect ➡️** | NEGATIVE ⬇️ |

**Mechanism:**
- Critical for **activating HEPN nuclease domains**
- Mutations here allow binding but prevent cleavage
- Interacts with Helical-1 domain for nuclease activation

---

## Summary Comparison

| Aspect | Proximal (1-11) | Distal (12-22) |
|--------|----------------|----------------|
| **Primary Role** | Nuclease activation | Target recognition/binding |
| **Binding Sensitivity** | Low (tolerant) | High (sensitive) |
| **Cleavage Sensitivity** | High | High |
| **Structure Tolerance** | High | Low |
| **Mismatch Tolerance** | High for binding | Low |

---

## Interpretation for CSV Data

Your CSV contains:
1. **Sequence** (75nt total)
2. **Structure prediction** (dot-bracket notation)
3. **MFE** (Minimum Free Energy in kcal/mol)
4. **Binding affinity** (normalized ΔABA value)

### Structure Column Interpretation:
- `.` = unpaired nucleotide
- `(` and `)` = base paired nucleotides

**Key principle:**
- **More parentheses in positions 42-52 (distal)** → **Lower binding affinity**
- Parentheses in positions 31-41 (proximal) → No effect on binding

### Example:
```
Sequence: ...GCATAGAGAGGTTATCCGCTTC...
Structure: ...((((((...((((((........))..))))..))..))))...
                    ↑                    ↑
                Proximal            Distal (many pairs)
                (no effect)         (NEGATIVE effect)
```

---

## Practical Applications

### For Binding Prediction:
1. **Focus on distal region (positions 42-52)** in your sequences
2. Count base pairs in structure prediction for this region
3. More structure → Expect lower binding affinity

### For CRISPR Design:
1. **Avoid structured distal regions** for high-affinity binding
2. **Use proximal mismatches** to reduce cleavage while maintaining binding
3. **Use distal mismatches** to discriminate between variants (SNP detection)

---

## Detailed Quantitative Results

### 1. Position-Specific Mismatch Effects

| Position in Target | Position in Full Sequence | Effect on Binding (ΔABA) | Notes |
|-------------------|---------------------------|-------------------------|--------|
| 1-12 | 31-42 | **Minimal/No effect** | Tolerates single mismatches |
| 13-22 | 43-52 | **Significant decrease** | Highly sensitive to mismatches |

**Specific Examples:**
- **Position 21 (Full seq: 51)**
  - C→U substitution: Similar to matched target (G-U wobble tolerated)
  - C→G substitution: Binding virtually undetectable (~3+ fold decrease)

### 2. PFS (Flanking Sequence) Effects - Quantified

> **PFS = Protospacer Flanking Sequence:** The nucleotides immediately adjacent to (flanking) the target sequence. In your data, these are the 3 nucleotides before and after the 22nt variable target region. Some CRISPR enzymes require specific PFS sequences for recognition, but Cas13d does not have a strong PFS requirement.

> **Note on "Fold Change":** Higher fold change = **stronger binding** = **lower Kd** = **higher affinity**. 
> - "2-fold stronger" means the binding affinity is 2× better (Kd is 2× lower)
> - "3-fold decrease" means binding affinity is 3× worse (Kd is 3× higher)

> **Note on PFS mechanism:** The 3nt PFS sequences (at positions 28-30 for 5'-PFS, 53-55 for 3'-PFS) affect RNA secondary structure, which then exposes or occludes nucleotides in the target region.

| PFS Type | PFS Position | Example | Fold Change | Mechanism | Target Positions Exposed/Affected |
|----------|-------------|---------|-------------|-----------|----------------------------------|
| **5'-PFS** | 28-30 | GUA vs UAA | **~2-fold** stronger (GUA) | GUA forms stem with constant region, exposing target positions 19-22 | Target: 19-22 (Full seq: 49-52) |
| **3'-PFS** | 53-55 | GCU vs UGG | **~2-fold** stronger (GCU) | GCU forms stem with constant region, exposing target positions 14-20 | Target: 14-20 (Full seq: 44-50) |
| **Overall PFS range** | 28-30 & 53-55 | All combinations | **~3-fold** difference max | Weak direct PFS preference; effect mainly through structure | N/A |

### 3. RNA Secondary Structure Impact (Statistical)

**Distal Region (12-22, Full seq: 42-52):**

| # Base Pairs in Distal | Effect on Binding | Statistical Significance |
|------------------------|------------------|-------------------------|
| 0-3 pairs | High binding | Baseline |
| 4-6 pairs | Moderate decrease | p < 0.001 |
| 7-9 pairs | Strong decrease | p < 1e-08 |
| 10+ pairs | Very low/undetectable | p < 1e-22 |

**Proximal Region (1-11, Full seq: 31-41):**
- No significant correlation between base pairs and binding (p > 0.05)

### 4. Insertions & Deletions Effects

| Location | Position in Target | Position in Full Seq | Effect on Binding |
|----------|-------------------|---------------------|------------------|
| **Proximal** | 1-11 | 31-41 | High affinity retained |
| **Distal** | 12-22 | 42-52 | Not tolerated (strong decrease) |

**Specific Example (from paper):**
- **C insertion between 19-20** (Full seq: 49-50): Reduces base pairs at 13-20 → **Increases** ΔABA vs matched target
- **G insertion at same position**: Creates new base pairs → Undetectably low binding

### 5. Binding Kinetics (BLI Measurements)

| Parameter | Low-affinity PFS | High-affinity PFS | Interpretation |
|-----------|-----------------|-------------------|----------------|
| **ka (on-rate)** | Slower | Faster | Structure affects initial recognition |
| **kd (off-rate)** | Similar | Similar | Duplex stability similar once formed |
| **Kd (affinity)** | Higher (weaker) | Lower (stronger) | Dominated by on-rate differences |

### 6. Double Mismatch Effects

| Mismatch Locations | Effect on Binding | Notes |
|-------------------|------------------|--------|
| Both in proximal (1-12) | Minimal effect | Dark blue in Fig 3C |
| One or both in distal (13-22) | Strong decrease | Yellow/red in Fig 3C |
| **C21U + A20G** | Undetectable | Creates base pairs at 19-21 (distal) |
| **C21U + any proximal** | Similar to matched | Structure remains favorable |

### 7. SARS-CoV-2 Variant Discrimination

| crRNA Design | SNP Position | Position Type | Binding Difference | Cleavage Discrimination |
|-------------|--------------|---------------|-------------------|----------------------|
| **crRNA-1** | 17 (distal) | Full seq: 47 | ~5-fold weaker for variant | ~5-fold reduction |
| **crRNA-2** | 1 (proximal) | Full seq: 31 | Comparable binding | ~5-fold reduction |

**Key insight:** Both designs discriminate variants, but through different mechanisms (binding vs cleavage).

## Model Performance

The paper developed 6 predictive models. **Model VI** performed best:
- **Pearson's r = 0.76**
- Combines: intramolecular base pairing + relative encoding + MFE

### Model Components Performance:

| Model | Components | Pearson's r | Notes |
|-------|-----------|-------------|--------|
| I | Base pairing only | 0.51 | Structure matters but insufficient |
| II | Base pairing + MFE | ~0.51 | MFE adds minimal improvement |
| III | Relative encoding only | 0.65 | Better than structure alone |
| IV | Relative encoding + MFE | ~0.65 | MFE minor improvement |
| V | Base pairing + Relative encoding | 0.75 | Best combination |
| VI | All three | **0.76** | Marginal improvement from MFE |
| ML (CNN) | Machine learning | **0.77** | Only slightly better |

**Key features from Model VI:**
- Distal base pairing penalty: **HIGH**
- Proximal base pairing penalty: **LOW** 
- Distal mismatch penalty: **HIGH** (positions 13-22)
- Proximal mismatch penalty: **MINIMAL** (positions 1-12)

---

## Reference

**Paper:** Kuo et al. (2024) "Massively parallel profiling of RNA-targeting CRISPR-Cas13d"  
*Nature Communications* 15, 498  
DOI: 10.1038/s41467-024-44738-w

**Key Innovation:** RNA-CHAMP platform - massively parallel profiling of >10,000 RNA variants to understand Cas13d binding specificity