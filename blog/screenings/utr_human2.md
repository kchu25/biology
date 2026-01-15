@def title = "Human 5' UTR to Translation: How They Measured It"
@def published = "15 January 2026"
@def tags = ["rna", "screening"]

# Human 5' UTR to Translation: How They Measured It

**Reference:** Sample, P. J., Wang, B., Reid, D. W., Presnyak, V., McFadyen, I., Morris, D. R., & Seelig, G. (2019). Human 5′ UTR design and variant effect prediction from a massively parallel translation assay. *Nature Biotechnology*, *37*(7), 803–809. https://doi.org/10.1038/s41587-019-0164-5

---

## The Big Picture

This is a **sequence-to-translation task** for human cells! They're mapping:
- **Input**: 50-nucleotide 5' UTR sequences  
- **Output**: Ribosome loading (translation activity)

Think of it as: *How many ribosomes are actively translating this mRNA?*

---

## Key Difference from the Yeast Study

> **Why NOT use growth selection like in yeast?**
>
> The yeast study used growth selection because they could link UTR activity to cell survival. But for human cells, they wanted to:
> - Separate **transcription** effects from **translation** effects
> - Test RNA directly without needing DNA in the nucleus
> - Measure translation of many different UTRs in a **single cell** simultaneously
>
> **Solution**: Polysome profiling!
> - Inject pre-made mRNA directly into cells (bypasses transcription)
> - Each cell gets a mix of ~hundreds of different mRNA variants
> - Measure how many ribosomes load onto each mRNA type

---

## The Measurement Pipeline

### 1. **The Setup**

They created ~280,000 random 50-nt sequences as 5' UTRs for eGFP:

```
[25 nt defined] + [50 nt random] → ATG → eGFP gene
```

The defined 25 nt at the start is just for PCR amplification later (like a barcode handle).

### 2. **What is Polysome Profiling?**

> **Polysome Profiling 101**
>
> Imagine ribosomes as trains on a track (the mRNA). A "polysome" is when multiple ribosomes are on the same mRNA simultaneously, like a train convoy.
>
> **The key idea:**
> - More ribosomes on an mRNA = more active translation
> - Polysomes are physically heavy (lots of ribosome protein!)
> - You can separate them by weight using a density gradient
>
> **How it works:**
> 1. Freeze ribosomes in place (add cycloheximide drug)
> 2. Break open cells, extract all mRNA+ribosome complexes
> 3. Layer the extract on top of a sucrose gradient (like a density smoothie: 20% → 55% sucrose)
> 4. Spin in ultracentrifuge for 3 hours at 151,000 × g
> 5. Heavy complexes (many ribosomes) sink far down
> 6. Light complexes (few ribosomes) stay near the top
> 7. Collect 14 fractions and sequence the mRNA in each

### 3. **The Experimental Workflow**

> **How Do They Measure Translation?**
>
> **Day 1 - Make the RNA:**
> 1. Take the plasmid library (DNA)
> 2. PCR amplify just the UTR + eGFP region, adding T7 promoter
> 3. Use *in vitro* transcription (IVT) to make mRNA from this DNA template
> 4. Add a 5' cap structure (like cells do naturally)
> 5. Add poly-A tail (70 nt)
> 6. Result: Library of ~280,000 different mRNA molecules!
>
> **Day 2 - Transfect human cells:**
> 1. Take HEK293T cells (human kidney cells)
> 2. Use lipid nanoparticles (Lipofectamine) to deliver mRNA into cells
> 3. Wait 12 hours for translation to happen
> 4. Each cell receives a cocktail of many different mRNA variants!
>
> **Day 3 - Polysome profiling:**
> 1. Add cycloheximide to freeze ribosomes on mRNA
> 2. Lyse cells and extract mRNA-ribosome complexes
> 3. Layer lysate on sucrose gradient
> 4. Ultracentrifuge for 3 hours
> 5. Collect 14 fractions from top (light) to bottom (heavy)
> 6. Extract RNA from each fraction
> 7. Reverse transcribe to cDNA
> 8. PCR amplify the UTR region
> 9. Deep sequence all fractions
>
> **The counting:**
> - Fraction 1 (top): mRNA with 0-1 ribosomes
> - Fraction 7 (middle): mRNA with ~4 ribosomes
> - Fraction 14 (bottom): mRNA with 8+ ribosomes
> 
> Count how many copies of each UTR variant appear in each fraction!

### 4. **The Key Measurement: Mean Ribosome Load (MRL)**

> **Critical Difference from Yeast Study:**
>
> **Yeast (Cuperus et al.)**: Used log₂ enrichment comparing before/after selection
> - Measured competitive fitness over time
> - Relative measurement (need a baseline population)
>
> **Human (Sample et al.)**: Uses Mean Ribosome Load (MRL) directly
> - Measures absolute ribosome occupancy per mRNA
> - No "before/after" comparison needed!
> - Each measurement is self-contained

For each 5' UTR sequence $s$, they calculate:

$\text{MRL}(s) = \sum_{i=1}^{14} \left( f_i(s) \times r_i \right)$

Where:
- $f_i(s)$ = **fraction** of sequence $s$ found in fraction $i$ (normalized by total reads in that fraction)
- $r_i$ = number of ribosomes associated with fraction $i$ (0, 1, 2, ..., 8+)

**Normalization per fraction:**

$f_i(s) = \frac{\text{count}_i(s)}{\sum_{s'} \text{count}_i(s')}$

This tells us: "Out of all the mRNA reads in fraction $i$, what proportion is sequence $s$?"

**Then we weight by ribosome number:**
- Sequence mostly in fraction 1 (no ribosomes) → low MRL
- Sequence mostly in fraction 14 (many ribosomes) → high MRL
- The sum gives us the average ribosome count per mRNA!

**What does MRL mean?**

| MRL Value | Interpretation |
|-----------|----------------|
| ~2 | Barely any ribosomes (poor translation) |
| ~5 | Moderate ribosome loading |
| ~8+ | Heavy ribosome loading (strong translation) |

**Intuition**: MRL is literally the **average number of ribosomes** sitting on an mRNA molecule of that sequence. If a UTR is good for translation, many ribosomes will pile onto that mRNA, creating heavy polysomes that sink to the bottom fractions!

---

## Key Advantages of This Approach

### Why polysome profiling beats growth selection for human cells:

1. **Separates transcription from translation**
   - They inject pre-made mRNA → bypasses the nucleus entirely
   - Measures ONLY translation, not gene expression
   
2. **Higher throughput**
   - Each cell translates hundreds of different mRNA variants
   - Don't need to transform cells with individual plasmids
   
3. **More direct measurement**
   - Literally counting ribosomes on each mRNA
   - Not relying on downstream phenotypes

4. **Works with modified RNA**
   - They tested pseudouridine (Ψ) and N1-methylpseudouridine (m1Ψ)
   - Important for mRNA therapeutics (like COVID vaccines!)

---

## Supporting Validation

### Flow Cytometry with eGFP
For 10 sequences, they measured actual eGFP fluorescence:

$$\text{Fluorescence} \propto \text{Protein expression}$$

Correlation with MRL: $R^2 = 0.87$ ✓

### Comparison with Published Data
Tested 77 UTRs from another study (Ferreira et al.):

Correlation: $R^2 = 0.73 - 0.85$ across 6 cell lines ✓

---

## The CNN Model Task

**Input**: One-hot encoded sequence matrix
- Shape: $(4 \times 50)$ for 50-nt UTRs
- Extended to $(4 \times 100)$ for variable-length model

**Output Options:**
1. **MRL model**: Single value predicting mean ribosome load
   - $\hat{y} \in \mathbb{R}$
   
2. **Polysome profile model**: 14 values predicting distribution across fractions
   - $\hat{y} = [\hat{f}_1, \hat{f}_2, ..., \hat{f}_{14}]$

**Loss function**: Mean squared error

$$\mathcal{L} = \frac{1}{N}\sum_{i=1}^{N}(y_i - \hat{y}_i)^2$$

**Model performance:**
- MRL prediction: $R^2 = 0.93$ on held-out random sequences
- Polysome profile: $R^2 = 0.83$ average across all fractions
- Human native UTRs: $R^2 = 0.81$ (35,212 sequences!)

---

## Applications

### 1. Engineering New UTRs
Using genetic algorithm + CNN:
- Design UTRs for targeted expression levels
- Successfully created sequences hitting MRL targets of 3, 4, 5, 6, 7, 8, 9

### 2. Variant Effect Prediction
Tested 3,577 disease-associated SNVs from ClinVar:
- Model explains 55% of variant effects overall
- **76% for high-impact variants** (log₂ change > 0.5)
- Identified 45 SNVs that substantially alter translation
- Example: rs867711777 in CPOX gene (hereditary coproporphyria)

### 3. Modified RNA (mRNA therapeutics)
Tested Ψ and m1Ψ modifications:
- Model trained on unmodified RNA: $R^2 = 0.68 - 0.76$ on modified RNA
- Retrained on modified RNA: improved significantly
- Shows how RNA modifications affect structure and translation

---

## Key Insight

MRL captures the **translation initiation and elongation efficiency**:
- How easily ribosomes find the start codon
- How many ribosomes can load simultaneously
- Impact of secondary structure
- Effect of upstream AUGs (uAUGs) and upstream ORFs (uORFs)

Unlike growth selection (yeast study), this **isolates translation** from:
- ❌ Transcription rate
- ❌ mRNA stability (partially)
- ❌ Protein function
- ❌ Protein stability

It's a more direct measure of the 5' UTR's role in translation! 🧬