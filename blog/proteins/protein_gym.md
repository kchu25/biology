@def title = "ProteinGym: A Massive Benchmark for Protein Fitness Prediction"
@def published = "14 November 2025"
@def tags = ["proteins", "mutegenisis"]

# ProteinGym: A Massive Benchmark for Protein Fitness Prediction

## What's This Paper About?

Imagine you're a protein engineer. You want to know: "If I mutate position 42 from Alanine to Glycine, will my protein still work?" 

There are millions of possible mutations, but testing them all in the lab is expensive and slow. So we need **machine learning models** to predict which mutations are good or bad.

But here's the problem: **there are tons of ML models out there** (ESM, ProGen2, AlphaFold-based methods, etc.), and it's hard to compare them fairly because:
- Everyone tests on different proteins
- Different metrics are used
- Small benchmarks don't tell the full story

**Enter ProteinGym:** A massive, standardized benchmark to finally answer: "Which models actually work best?"

---

## The Scale is Unprecedented

### Dataset Size

- **217 proteins** with substitution data (2.4M mutations tested!)
- **66 proteins** with insertion/deletion data (289k mutations)
- **Plus clinical data:** 65k human disease variants from ClinVar

For comparison, previous benchmarks had like 30-40 proteins with ~700k mutations total. This is 2-3 orders of magnitude larger.

### Diversity

The proteins come from:
- Humans, bacteria, viruses, other organisms
- Different functions: enzymes, receptors, structural proteins
- Different properties measured: stability, activity, binding, expression

### Models Tested

They evaluated **70+ models** including:
- Protein language models (ESM, ProGen2, RITA)
- Evolution-based models (EVmutation, DeepSequence, EVE)
- Structure-based models (ProteinMPNN, ESM-IF1)
- Hybrid approaches (Tranception, TranceptEVE)

---

## The Core Question: Zero-Shot vs. Supervised

This is where it gets interesting. There are **two fundamentally different ways** to predict mutation effects:

---

## Zero-Shot: Learning from Evolution (No Labels!)

### The Core Idea

**Zero-shot models never see fitness measurements during training.** Instead, they train on:
- Millions of unlabeled protein sequences from nature
- Multiple sequence alignments showing evolutionary variation
- 3D structures from the Protein Data Bank

The **key assumption:** If a sequence exists in nature, evolution has already "tested" it. Sequences that are common in nature = probably functional.

> ### But Wait - Can They Predict Anything Beyond "Natural Fitness"?
>
> **Short answer: Not really!** This is a crucial limitation.
>
> **What zero-shot models CAN predict:**
> - Properties that evolution directly selects for (stability, basic function)
> - Properties correlated with evolutionary fitness
> - General "will this protein fold and work?"
>
> **What zero-shot models CANNOT predict well:**
> - **Novel functions** not found in nature (e.g., binding to synthetic drugs)
> - **Exotic properties** like:
>   - Fluorescence brightness (GFP engineering)
>   - Specific binding to non-natural targets
>   - Activity in non-physiological conditions (industrial temps/pH)
>   - Repression vs activation (if evolution doesn't distinguish them)
> - **Trade-offs** between competing objectives (e.g., high activity but low stability)
> - **Context-specific fitness** that differs from natural environment
>
> **Example where zero-shot fails:**
> - Task: Engineer enzyme to work at 90°C for industrial biofuel production
> - Evolution optimized it to work at 37°C in E. coli
> - Zero-shot models predict mutations that maintain 37°C function
> - But you need something evolution never selected for!
>
> **That's why supervised learning matters:** 
> - You collect data in YOUR conditions (high temp, specific substrate, etc.)
> - Train on YOUR objective function
> - Now the model learns what YOU care about, not what evolution cared about
>
> **The paper shows this:** Supervised models (0.613 Spearman) significantly outperform zero-shot (0.456) when you have data, because they learn the specific property being measured, not just evolutionary plausibility.

### Three Types of Zero-Shot Models

#### 1. **Protein Language Models** (ESM, ProGen2, RITA)

**What they train on:**
- Millions of sequences like "MKDLSTVQAPL..." from UniRef databases
- No labels, no annotations - just raw sequences

**Training objective:**
- Predict masked amino acids (like BERT): "MKD_STVQ" → predict the blank
- Or predict next amino acid (like GPT): "MKD" → predict next is "L"

**What they learn:**
- Statistical patterns: "Which amino acid combinations are common?"
- Context: "If positions 10-20 form a helix, position 15 is probably hydrophobic"
- Global constraints: "Cysteines often come in pairs (disulfide bonds)"

**At test time:**
```
Wild-type: MKDLSTVQAPL  →  p(sequence) = 0.0042
Mutant:    MKDGSTVQAPL  →  p(sequence) = 0.0001

Score = log(0.0001 / 0.0042) = -3.6 (negative = bad mutation!)
```

The model thinks the mutant is much less likely to exist in nature → probably breaks the protein.

#### 2. **Alignment-Based Models** (EVmutation, DeepSequence, EVE)

**What they train on:**
- Multiple Sequence Alignments (MSAs) of homologous proteins
- Example: 10,000 sequences of kinase proteins from different species

**What they learn:**
- **Co-evolution patterns:** "If position 10 is Arginine, position 50 tends to be Aspartate (they form a salt bridge!)"
- **Epistasis:** How mutations interact with each other
- **Conservation:** Which positions never change (probably important!)

**Training objective:**
- Learn probability distribution $p(\text{sequence} | \text{family})$
- Capture pairwise dependencies between positions

**Example:**
```
Position:    10    50    72
Wild-type:   R     D     W     ← Common in MSA (high probability)
Mutant:      K     D     W     ← K at 10 breaks salt bridge with D at 50 (lower probability)
```

**Why this is different from language models:**
- Language models learn from ALL proteins (general patterns)
- Alignment models learn from ONE protein family (family-specific patterns)
- Alignment models explicitly model position interactions

#### 3. **Inverse Folding Models** (ProteinMPNN, ESM-IF1)

**What they train on:**
- 3D protein structures from PDB
- They see: structure coordinates + the sequence that folds into it

**The "inverse" problem:**
- **Normal folding:** sequence → structure (what AlphaFold does)
- **Inverse folding:** structure → sequence (what these do)

**Training objective:**
- Given a backbone structure, predict: $p(\text{sequence} | \text{structure})$
- Learn: "What amino acids fit well at each position given the 3D environment?"

**What they learn:**
```
Position in hydrophobic core  →  prefers V, L, I, F (hydrophobic)
Position on surface           →  prefers K, R, D, E (charged)
Position in beta sheet        →  prefers V, I, Y (beta-sheet formers)
```

**At test time:**
1. Get protein structure (from experiment or AlphaFold2 prediction)
2. Score wild-type: $p(\text{MKDLST...} | \text{structure})$
3. Score mutant: $p(\text{MKDGST...} | \text{structure})$
4. Compare: Does the mutant still fit the structure?

**Key limitation:** Only works for substitutions! Insertions/deletions change the backbone length, so you can't use the same structure.

---

### Wait, But What Are They Actually Predicting?

Here's the subtle part: **These models don't directly predict "fitness"** - they predict **evolutionary plausibility** or **structural compatibility**.

The magic is that these often correlate with functional properties!

| What Model Predicts | What Benchmark Measures | Does It Correlate? |
|---------------------|-------------------------|-------------------|
| Evolutionary probability | Enzyme activity | ✓ Usually yes |
| Evolutionary probability | Thermostability | ✓ Usually yes |
| Evolutionary probability | Binding affinity | ✓ Often yes |
| Structural compatibility | Expression level | ✓ Sometimes |

**Why does this work?**
- Unstable proteins get degraded by evolution
- Non-functional proteins don't help survival
- Structure determines function

**But it's not perfect:**
- Lab conditions ≠ natural environment
- Some assays measure properties evolution doesn't directly select for
- That's why correlation isn't 1.0!

### The Zero-Shot Scoring Formula

Most zero-shot models use this:

$$\text{Mutation Score} = \log \frac{p(x_{\text{mutant}})}{p(x_{\text{wt}})}$$

Where:
- $p(x)$ = probability model assigns to sequence $x$
- Positive score → mutant more likely than wild-type → probably good
- Negative score → mutant less likely → probably bad

**But there are variations:**
- Some normalize by sequence length
- Some use marginal probabilities at mutated positions only
- TranceptEVE combines multiple scoring schemes

---

## Supervised: Learning from Actual Fitness Data

Now let's talk about the **completely different approach:** supervised learning.

### The Core Idea

**Train on actual fitness measurements** from the target protein (or related proteins).

**Example:**
```
Training data for β-lactamase:
MKDLST... → fitness = 0.8
MKDGST... → fitness = 0.2
MKDAST... → fitness = 0.9
... (thousands more)

Now predict:
MKDRST... → fitness = ?
```

### Why Is This Different?

| Zero-Shot | Supervised |
|-----------|-----------|
| Learns from millions of proteins | Learns from one protein family |
| Never sees fitness labels | Directly trained on fitness labels |
| General patterns | Task-specific patterns |
| Works on any protein | Needs data for each protein |

### The Challenge: Overfitting

Here's the problem: **mutations aren't independent!**

If you train on mutation A42G and test on A42V:
- Both are at position 42
- Both change from Alanine to small amino acids
- The model might "cheat" by memorizing position 42 is important

**Solution: Careful cross-validation**

The paper uses 3 schemes:

1. **Random split:** Randomly assign mutations to train/test
   - Easiest task
   - Some position overlap between train and test

2. **Contiguous split:** Split sequence into segments
   ```
   Positions 1-50   → Train
   Positions 51-100 → Test
   Positions 101-150 → Train
   ...
   ```
   - Harder: Must predict on completely unseen positions

3. **Modulo split:** Assign positions by position % 5
   ```
   Positions 1,6,11,16... → Fold 1
   Positions 2,7,12,17... → Fold 2
   ...
   ```
   - Hardest: Forces model to generalize across the sequence

### Three Types of Supervised Models

#### 1. **One-Hot Encoding + Ridge Regression**

**Simple baseline:**
```
Input:  [0,1,0,...,0] (one-hot encoding of sequence)
        + zero-shot scores from ESM, Tranception, etc.
Output: predicted fitness

Model:  Linear regression with L2 penalty
```

**Idea:** Use zero-shot predictions as features, learn linear combination that best predicts fitness.

#### 2. **Embedding Models**

**Smarter approach:**
```
Input:  [2.4, -1.1, 0.8, ...] (embedding from protein language model)
        + zero-shot score
Output: predicted fitness

Model:  Ridge regression on embeddings
```

**Why better?** Language model embeddings capture complex patterns (secondary structure, hydrophobicity, etc.) that one-hot encoding misses.

#### 3. **ProteinNPT** (Best Supervised Model)

**Fancy architecture:**
- Non-parametric transformer that jointly models sequences AND labels
- Uses "axial attention" across both data dimensions
- Semi-supervised: Can use unlabeled sequences too

**Training objective:** 
- Predict fitness from sequence
- Reconstruct masked amino acids
- Learn rich representation of sequence-function relationship

---

## Key Results: Who Won?

### Zero-Shot Champions

**Winner: TranceptEVE** (Spearman correlation = 0.456)

What is it? A **hybrid model** that combines:
- Autoregressive language model (Tranception)
- Evolutionary priors from alignment models (EVE)
- MSA retrieval at inference time

**Why did it win?**
- Gets general patterns from language model
- Gets family-specific patterns from alignment
- Best of both worlds!

**Runners-up:**
- GEMME (alignment-based) - surprisingly good!
- VESPA (language model + conservation)
- ESM-IF1 (inverse folding) - best on high-quality structures

### Supervised Champions

**Winner: ProteinNPT** (Spearman correlation = 0.613)

Much better than zero-shot! But requires training data from the target protein.

**Performance by cross-validation:**
- Random split: 0.730 (easiest)
- Modulo split: 0.564 (harder)
- Contiguous split: 0.547 (hardest)

**Key insight:** Big gap between random and positional splits → models rely heavily on learning which positions are important.

### Interesting Findings

1. **Autoregressive > Masked models** for zero-shot
   - ProGen2, Tranception > ESM models
   - Why? Autoregressive explicitly models sequence dependencies

2. **Alignment models still competitive**
   - GEMME nearly ties with TranceptEVE
   - Especially good for viral proteins and low MSA depth

3. **Different models for different tasks**
   - ESM models: better Spearman (good at ranking all mutations)
   - Alignment models: better NDCG (better at finding top mutations)
   - Matters for protein design vs. variant interpretation!

4. **Structure helps when available**
   - ESM-IF1 gets 0.422 Spearman
   - But needs good structure (experimental or AlphaFold2)

---

## What Does "Fitness" Even Mean?

Here's something subtle: **"Fitness" depends on the assay!**

The benchmark measures 5 different properties:

| Property | What It Measures | Example Assay |
|----------|------------------|---------------|
| **Activity** | Enzymatic/biochemical function | β-lactamase antibiotic resistance |
| **Binding** | Affinity to target | ACE2 binding to SARS-CoV-2 spike |
| **Stability** | Thermostability, folding | Melting temperature |
| **Expression** | Protein abundance | Cell surface display |
| **Organismal Fitness** | Cell growth/survival | Yeast growth rate |

**The models don't know what they're predicting!** They just predict:
- "Is this mutation evolutionarily plausible?" (zero-shot)
- "Based on training data, what's the fitness?" (supervised)

Then we check: Does evolutionary plausibility correlate with the measured property?

**Answer:** Usually yes, but it varies:
- Best correlation: Stability, Activity
- Moderate correlation: Binding, Expression
- Variable: Organismal fitness (depends on environment)

---

## Clinical Applications

The paper also tests on **human disease variants** from ClinVar:
- 63k substitutions across 2,525 genes
- Task: Predict pathogenic vs. benign

**Interesting result:**
- Best supervised model (ClinPred): AUC = 0.981
- Best zero-shot model (TranceptEVE): AUC = 0.920

**But:** Supervised models trained on ClinVar → potential circularity!
- Zero-shot models are more "honest" predictors
- Still achieve 0.92 AUC without ever seeing clinical labels

**Why care?**
- Clinical guidelines hesitate to use ML predictions
- This shows zero-shot models can achieve good performance without bias from training labels

---

## Why This Matters

### Before ProteinGym:
- 30-40 protein benchmarks
- Inconsistent evaluation
- Hard to compare different model types
- Mostly substitutions only

### After ProteinGym:
- 250+ proteins, 2.7M mutations
- Standardized metrics
- Direct comparison of alignment-based, language models, inverse folding
- Both substitutions AND indels
- Both experimental AND clinical data

### Practical Impact:

1. **Model selection guidance:** 
   - Low MSA depth? Use language models
   - Good structure? Try inverse folding
   - Need top designs? Use alignment models (better NDCG)

2. **Reproducibility:**
   - All code, data, predictions public
   - Interactive website: proteingym.org

3. **Future benchmark:**
   - Will be updated with new assays and models
   - Community standard for evaluation

---

## The Big Picture

This paper doesn't invent new models - it creates the **benchmark infrastructure** to fairly compare them.

Think of it like:
- **ImageNet** for computer vision
- **GLUE/SuperGLUE** for NLP
- **CASP** for protein structure prediction

Now protein fitness prediction has **ProteinGym**.

**Key philosophical point:** Zero-shot and supervised aren't competing - they're complementary!
- Zero-shot: When you have no data for your protein
- Supervised: When you have some data and want to optimize further

The best approach often combines both (like TranceptEVE does).

---

## Resources

Everything is open source:
- GitHub: https://github.com/OATML-Markslab/ProteinGym
- Website: https://www.proteingym.org
- All datasets, MSAs, structures, predictions available
- Unified codebase to run all 70+ models