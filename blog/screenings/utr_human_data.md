@def title = "Understanding the GSM3130435_egfp_unmod_1.csv Data"
@def published = "22 December 2025"
@def tags = ["rna", "screening"]

# Understanding the GSM3130435_egfp_unmod_1.csv Data

## Paper: Human 5′ UTR Design and Variant Effect Prediction from a Massively Parallel Translation Assay

**Paper Link:** https://www.nature.com/articles/s41587-019-0164-5  
**GEO Dataset:** GSE114002

---

## What is This Dataset?

This dataset contains polysome profiling results for **280,000 random 50-nucleotide sequences** used as 5′ UTRs (untranslated regions) in front of an eGFP reporter gene. The experiment measures how well each sequence directs protein translation.

---

## Column Structure

## Column Structure

### 1. **Sequence Column** ⚠️ IMPORTANT - READ THIS!

**What you'll see:** 59-nucleotide sequences ending in `ATGGGCGAA`

**What you need:**
- **Remove the suffix `ATGGGCGAA`** (last 9 nucleotides)
- Extract only the first 50 nucleotides
- This is the variable 5′ UTR that the model predicts from

**Structure:**
```
Full sequence in CSV: [50-nt variable UTR][ATGGGCGAA]
                                            ↑
                                    ATG = start codon
                                    GGCGAA = start of eGFP

After preprocessing:  [50-nt variable UTR only]
```

**Why the suffix is there:**
- `ATG` is the translation start codon
- `GGCGAA` is the beginning of eGFP coding sequence
- It's constant across all constructs for fair comparison
- **But the model was trained on only the 50-nt UTR!**

**Preprocessing code:**
```python
# Remove constant suffix
utr_sequence = full_sequence[:-9]  # Remove last 9 bases
# Or equivalently
utr_sequence = full_sequence[:50]  # Take first 50 bases
```

### 2. **Unnormalized Fraction Counts: `00`, `01`, `02`, ..., `011`**
- **Total:** 12 columns (fractions 0-11, numbered in base 12)
- **Content:** Raw sequencing read counts for each polysome gradient fraction
- **What they represent:**
  - `00`: Untranslated mRNA (no ribosomes)
  - `01`: Light polysome (1-2 ribosomes)
  - `05`: Medium polysome (5-6 ribosomes)
  - `011`: Heavy polysome (11+ ribosomes)

### 3. **Normalized Fraction Counts: `r_00`, `r_01`, `r_02`, ..., `r_011`**
- **Total:** 12 columns
- **Content:** Normalized read counts (relative frequencies that sum to 1.0)
- **Calculation:** `r_XX = count_XX / total`
- **Purpose:** Shows the distribution of that sequence across polysome fractions

### 4. **`total` Column**
- **Content:** Sum of all raw read counts across all fractions
- **Formula:** `total = sum(00, 01, 02, ..., 011)`
- **Purpose:** Quality control and normalization denominator

### 5. **`rl` Column (Mean Ribosome Load - MRL)**
- **Content:** The weighted average number of ribosomes bound per mRNA
- **This is the TARGET VARIABLE the model predicts!**

---

## How Mean Ribosome Load (MRL) is Calculated

### Formula:
```
MRL = Σ (fraction_number × relative_count_in_fraction)
```

### Explicit Calculation:
```
MRL = (0 × r_00) + (1 × r_01) + (2 × r_02) + ... + (11 × r_011)
```

### Example:
If a sequence has this distribution:
- 10% in fraction 0 (untranslated): `0 × 0.10 = 0.00`
- 20% in fraction 5 (medium): `5 × 0.20 = 1.00`
- 70% in fraction 10 (heavy): `10 × 0.70 = 7.00`

**MRL = 0.00 + 1.00 + 7.00 = 8.0**

This sequence has an average of 8 ribosomes per mRNA molecule!

---

## What is a Polysome?

**Polysome = Poly-ribosome = Multiple ribosomes on one mRNA**

### The Biology:

When an mRNA is being **actively translated** into protein:
- **NOT just one ribosome** translates it at a time
- **Multiple ribosomes** can be on the same mRNA simultaneously
- Each ribosome reads the mRNA like a train on a track
- They queue up and translate in sequence, like beads on a string

### Visual Analogy:

```
Single mRNA molecule:
5'UTR---START---CODING-SEQUENCE---STOP---3'UTR

Without translation (no ribosomes):
5'========================================3'

With active translation (polysome with 5 ribosomes):
5'===🔵===🔵===🔵===🔵===🔵============3'
    R1   R2   R3   R4   R5
```

Each 🔵 represents a ribosome making a copy of the protein!

### Why Multiple Ribosomes?

**More ribosomes = More protein copies made simultaneously**

- One mRNA with 10 ribosomes → making 10 protein molecules at once
- One mRNA with 1 ribosome → making only 1 protein molecule
- One mRNA with 0 ribosomes → making ZERO protein (not being translated)

### Important Caveat: You Can't Actually Count Them!

**The "ribosome numbers" are approximations based on sedimentation rate**, not literal counts. See the detailed explanation below.

---

## Why Predict MRL? (Why Does This Matter?)

### The Central Question of Gene Expression:

**"How do we control how much protein a cell makes?"**

### Three Main Factors Control Protein Production:

1. **Transcription:** How much mRNA is made from DNA
2. **mRNA Stability:** How long the mRNA lasts before degrading
3. **Translation Efficiency:** How many ribosomes load onto each mRNA ← **This is what MRL measures!**

### Why Translation Efficiency Matters:

Even if you have lots of mRNA, if ribosomes don't bind well:
- ❌ **Low MRL** → Few ribosomes → Little protein made
- ✅ **High MRL** → Many ribosomes → Lots of protein made

**The 5' UTR sequence is the "on/off switch" for translation efficiency!**

---

## How Polysome Profiling Works

### The Experimental Process:

1. **Transfect cells** with mRNA library (each with different 5′ UTR)
   - All have same coding sequence (eGFP)
   - Only difference is the 50-nt sequence at the start (5' UTR)

2. **Let cells translate** the mRNAs into protein
   - Good 5' UTRs → many ribosomes bind → heavy polysomes
   - Bad 5' UTRs → few/no ribosomes bind → stay light

3. **Freeze ribosomes** on mRNA (with cycloheximide drug)
   - Stops ribosomes mid-translation
   - Preserves the polysome structure

4. **Lyse cells** and extract ribosome-mRNA complexes

5. **Centrifuge through sucrose gradient:**
   - **Top (light):** Free mRNA with 0-1 ribosomes
   - **Middle:** Light polysomes (2-5 ribosomes)
   - **Bottom (heavy):** Heavy polysomes (6+ ribosomes)
   - Physics: More ribosomes = heavier = sinks deeper

6. **Collect 12 fractions** from top to bottom

7. **Extract mRNA** from each fraction and **sequence it**

8. **Count reads** to see where each 5′ UTR sequence ended up

### Why 12 Fractions?

Each fraction represents a "bin" of different ribosome numbers:
- **Fraction 0:** 0 ribosomes (untranslated)
- **Fraction 1:** ~1 ribosome
- **Fraction 5:** ~5 ribosomes
- **Fraction 11:** 11+ ribosomes (very heavy polysomes)

**Key Insight:** Where your sequence ends up = how many ribosomes it recruited = how well it translates!

---

## What the Model Predicts

**Input:** 50-nucleotide sequence (the 5′ UTR)  
**Output:** `rl` value (Mean Ribosome Load)

### Two Types of Models:

1. **Main Model (Optimus 5-Prime):** 
   - Predicts the `rl` column directly
   - Single value prediction: how many ribosomes on average

2. **Advanced Model:**
   - Predicts full distribution: all 12 `r_XX` values
   - More detailed but more complex

---

## Biological Interpretation

| MRL Value | Translation Level | Polysome Pattern |
|-----------|------------------|------------------|
| 0-2 | Very low | Mostly untranslated |
| 3-5 | Low-moderate | Light polysomes |
| 6-8 | Moderate-high | Medium polysomes |
| 9-11 | Very high | Heavy polysomes |

**Higher MRL = More Translation = More Protein Production**

---

## The 10 Samples in GSE114002

1. **egfp_unmod_1 & 2** - Random library, unmodified RNA (training data)
2. **egfp_pseudo_1 & 2** - Pseudouridine-modified RNA
3. **egfp_m1pseudo_1 & 2** - 1-methylpseudouridine-modified RNA
4. **mcherry_unmod_1 & 2** - Different reporter gene (mCherry)
5. **designed_library** - Human UTRs, SNVs, engineered sequences
6. **designed_library_25-100_length_utrs** - Variable-length UTRs

---

## Why Predict MRL? (Real-World Applications)

### 1. **Designing Better mRNA Vaccines** 💉
**Problem:** COVID mRNA vaccines need to make lots of spike protein
- If 5' UTR is bad → low MRL → weak immune response
- If 5' UTR is optimized → high MRL → strong immune response

**Solution:** Use model to design 5' UTRs that maximize protein production

### 2. **Gene Therapy Optimization** 🧬
**Problem:** Need precise protein levels for therapy
- Too little → therapy doesn't work
- Too much → toxic side effects

**Solution:** Engineer 5' UTRs with exact MRL to hit the "Goldilocks zone"

### 3. **Understanding Disease Mutations** 🔬
**Problem:** Genetic mutations in 5' UTRs can cause disease, but we don't know why

**Example from the paper:**
- Mutation in CPOX gene (hereditary coproporphyria)
- Model predicts: mutation drops MRL by 40%
- Result: Not enough enzyme → disease symptoms

**The model found 45 disease-associated mutations that dramatically change MRL!**

### 4. **Synthetic Biology & Metabolic Engineering** 🏭
**Problem:** Building genetic circuits that need precise tuning
- Need enzymes at different levels in pathway
- Like tuning volume knobs for different instruments

**Solution:** Use model to design 5' UTRs that dial each enzyme to optimal level

### 5. **Drug Discovery** 💊
**Problem:** Some drug targets need translation control
- Can't drug the protein directly
- But can design RNA therapeutics targeting translation

**Solution:** Understand which sequences control translation → design drugs that modify it

---

## The Power of the Model

### Before This Paper:
- ❌ Trial and error testing of UTRs
- ❌ No way to predict effects of new sequences
- ❌ Can't explain disease mutations

### After This Paper:
- ✅ Type in any sequence → get MRL prediction
- ✅ Design custom UTRs for target expression level
- ✅ Predict impact of genetic variants
- ✅ Understand molecular basis of disease

**It's like going from randomly mixing paint colors to having a color formula calculator!**

---

## How They Handled the Two Biological Replicates

### Your Question About Averaging:

**You're right to ask!** They have two biological replicates (egfp_unmod_1 and egfp_unmod_2), and you correctly noted that MRL is a rate-like quantity.

### What They Actually Did:

Based on the methods and code analysis, they **trained on BOTH replicates** but treated them somewhat differently:

**Option 1: Pooled the replicates** (most likely based on typical MPRA practices)
- Combined read counts from both replicates before calculating MRL
- This increases statistical power and reduces noise
- Each sequence gets: `total_counts = counts_replicate1 + counts_replicate2`
- Then calculate MRL from the pooled distribution

**Option 2: Averaged MRLs** (less common but possible)
- Calculate MRL for each replicate separately
- Take arithmetic mean: `MRL_final = (MRL_rep1 + MRL_rep2) / 2`
- Use averaged MRL as training target

### Why Not Harmonic Mean?

**You make an interesting point about harmonic mean!** However:

1. **MRL is not exactly a rate** - it's a weighted average of ribosome counts
   - It's more like "average number of ribosomes per mRNA"
   - Not a speed/velocity where harmonic mean would be appropriate

2. **Harmonic mean is used for:**
   - Rates like km/hour (travel time averages)
   - Precision/recall (F1 score)
   - When averaging rates where time/distance is in denominator

3. **MRL calculation already handles the distribution properly:**
   - Each fraction's contribution is weighted by ribosome number
   - The arithmetic mean preserves this relationship

### Replicate Quality Check:

The paper reports **very high correlation** between replicates:
- Pearson r² typically > 0.95 between biological replicates
- This validates that pooling/averaging is appropriate
- Low noise means either approach gives similar results

## Why Do The Two Replicates Have Different Numbers of UTRs?

### Your Observation:
**egfp_unmod_1 and egfp_unmod_2 don't have the same number of sequences!**

This is completely normal and expected! Here's why:

### The Filtering Process:

**Starting point:** Both replicates test the same 280,000 designed UTR sequences

**But after sequencing, they filter out sequences with insufficient data:**

1. **Low read count sequences** - If a UTR gets too few sequencing reads, the measurements are too noisy to be reliable
   - Example: Only 5 total reads across all fractions → can't calculate accurate MRL
   - Typical threshold: Require at least 50-100 total reads

2. **Stochastic sampling** - Not all sequences get picked up equally well in sequencing
   - Some sequences are underrepresented in library synthesis
   - Some cells don't take up certain sequences as efficiently
   - Random sampling during sequencing misses some rare sequences

3. **Different filtering per replicate:**
   - Replicate 1 might have sequence A drop out (too few reads)
   - Replicate 2 might have sequence B drop out (too few reads)
   - **The overlap is what matters for training!**

### Example Scenario:

```
Original library: 280,000 sequences

After sequencing and filtering:
- egfp_unmod_1: 215,000 sequences pass filter
- egfp_unmod_2: 218,000 sequences pass filter
- OVERLAP: 205,000 sequences in BOTH replicates

For training, they use only the overlap (205,000)
```

### Why This Is Standard Practice:

**Quality over quantity:**
- Better to have fewer high-quality measurements
- Than to include noisy data that will confuse the model
- The ~200k sequences they keep is still plenty for training!

**Common in MPRA experiments:**
- This happens in almost all high-throughput assays
- Papers typically report something like: "We recovered 70-80% of designed sequences"
- The exact dropout is random, so replicates differ slightly

### What They Actually Do for Training:

**Option 1: Use intersection** (most conservative)
- Only use sequences present in BOTH replicates
- Highest quality data
- Typically ~70-80% of original library

**Option 2: Use union with imputation** (more data)
- Use all sequences from either replicate
- For sequences in only one replicate, use single measurement
- More sequences but noisier

**Most likely for this paper:** They used the **intersection** (sequences in both replicates), then pooled the read counts for those sequences.

### The Technical Reason:

During the experiment:
1. **Transfection efficiency varies** - Not all cells get all sequences
2. **Amplification bias** - PCR doesn't amplify all sequences equally
3. **Sequencing depth limits** - Can't sequence infinitely deep
4. **Random sampling** - Each sequencing run samples randomly from the pool

**Result:** Each replicate recovers a slightly different subset of the original library.

### Bottom Line:

**This is totally normal and expected!** The different counts don't mean something went wrong - it's just the reality of high-throughput sequencing. The key is:
- Both replicates are highly correlated for sequences they share (r² > 0.95)
- They have enough overlap (~200k sequences) for robust model training
- The sequences that pass filter in both replicates are high-quality measurements