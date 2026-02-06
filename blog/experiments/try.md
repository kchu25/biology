@def title = "Experimental Validation of Predicted Motif Interactions"
@def published = "6 February 2026"
@def tags = ["experimental-biology"]

# Experimental Validation of Predicted Motif Interactions

## 1. Overview

The paper predicts two classes of results that are experimentally testable:

1. **Significant motifs**: cis-regulatory elements with Banzhaf indices significantly different from random coalitions (Section S1 of the supplementary).
2. **Motif interactions**: pairs and triplets of motifs whose co-occurrence produces non-additive effects on gene expression, quantified by interaction coefficients $\beta_{12}$ or $\beta_{123}$ (Section S2 of the supplementary).

Validating these predictions requires **reporter assays** that systematically perturb motif combinations and measure the resulting changes in gene expression or mRNA stability. The core experimental logic mirrors the statistical model from Equation (2) of the paper:

$$y = \beta_1 x_1 + \beta_2 x_2 + \beta_{12} x_1 x_2$$

where each construct corresponds to a specific $(x_1, x_2)$ configuration, and the measured reporter output serves as $y$.

---

## 2. Experimental Design

### 2.1 The Fundamental Logic

For each predicted interaction (say, between motif $m_1$ and motif $m_2$), we need **four constructs**:

| Construct | $x_1$ | $x_2$ | What it tests |
|-----------|--------|--------|---------------|
| Wild-type (both motifs intact) | 1 | 1 | $\beta_1 + \beta_2 + \beta_{12}$ |
| $m_1$ mutated | 0 | 1 | $\beta_2$ alone |
| $m_2$ mutated | 1 | 0 | $\beta_1$ alone |
| Both mutated | 0 | 0 | Baseline (no motif contribution) |

The interaction coefficient is then:

$$\hat{\beta}_{12} = y_{11} - y_{10} - y_{01} + y_{00}$$

If $\hat{\beta}_{12} > 0$: synergy (matches a positive $\beta_{12}$ prediction).  
If $\hat{\beta}_{12} < 0$: antagonism (matches a negative $\beta_{12}$ prediction).  
If $\hat{\beta}_{12} \approx 0$: additive (no interaction).

For **triplet interactions**, we need $2^3 = 8$ constructs (all combinations of three motifs present/absent).

### 2.2 Selecting Candidates

From the paper's results, prioritize interactions that are:

- **Statistically strong**: lowest adjusted $p$-values (well below the $\alpha = 10^{-10}$ threshold)
- **Biologically large**: largest $|\beta_{12}|$ or $|\beta_{123}|$ values
- **Organism-appropriate**: E. coli interactions are easiest to validate; yeast is intermediate; human UTR requires mammalian cell culture

A reasonable starting point is **3–5 pairwise interactions in E. coli** with the largest effect sizes.

---

## 3. Detailed Procedure: Molecular Cloning–Based Reporter Assay

The procedure below is conceptually similar to [standard molecular cloning workflows](https://kchu25.github.io/biology/blog/general/cloning/), but adapted for regulatory element testing. The key difference is that instead of cloning a gene of interest, we are cloning **synthetic regulatory sequences** upstream (or in the UTR) of a reporter gene.

### 3.1 Step-by-Step Protocol

#### Step 1: Design Synthetic Regulatory Sequences

For each predicted interaction, design oligonucleotides (~80–200 bp) containing:

- The **wild-type** regulatory region with both motifs
- **Mutant variants** where each motif is disrupted by point mutations (typically 3–5 bp changes in the core positions)
- Flanking sequences with restriction sites or homology arms for cloning

**Key design considerations:**
- Mutations should disrupt the motif's information content (change high-information-content positions in the PWM) without altering spacing between motifs
- Include a **scrambled control** where the entire regulatory region is randomized
- For UTR experiments, mutations should avoid introducing premature stop codons or strong secondary structures

**How to obtain the oligos:**
- Order as synthetic oligonucleotides from **IDT** (Integrated DNA Technologies), **Twist Bioscience**, or **Genewiz/Azenta**
- For sequences >200 bp, order as **gene fragments** (gBlocks from IDT, ~\$0.07–0.10/bp)
- Typical cost: \$10–\$90 per construct

#### Step 2: Choose a Reporter Vector

The reporter vector is the backbone plasmid into which you insert the regulatory sequences. This is analogous to choosing the destination vector in a standard cloning workflow.

| Organism | Recommended vector | Reporter | Selection |
|----------|-------------------|----------|-----------|
| E. coli (promoter) | pUA66 (from Zaslaver library) | GFPmut2 | Kanamycin |
| E. coli (promoter) | pSB1C3 (BioBrick) | GFP or RFP | Chloramphenicol |
| Yeast (promoter) | pRS416-CYC1 | yEGFP or LacZ | URA3 (auxotrophic) |
| Yeast (UTR) | pYES2 derivative | GFP/Luciferase | URA3 |
| Human (UTR) | psiCHECK-2 (Promega) | Renilla + Firefly luciferase | Ampicillin (bacterial) |
| Human (UTR) | pmirGLO (Promega) | Firefly + Renilla luciferase | Ampicillin |

**Source:** Order from **Addgene** (~\$75/plasmid) or request from collaborating labs.

#### Step 3: Cloning — Inserting Regulatory Sequences into the Reporter Vector

This is the core molecular cloning step, directly analogous to the [cloning workflow](https://kchu25.github.io/biology/blog/general/cloning/). There are two main approaches:

##### Option A: Restriction Enzyme Cloning (Traditional)

1. **Digest** the reporter vector with two restriction enzymes that flank the insertion site (e.g., EcoRI + BamHI for pUA66)
2. **Digest** (or design with matching overhangs) the synthetic regulatory fragment with the same enzymes
3. **Ligate** the fragment into the cut vector using T4 DNA ligase
4. **Transform** into competent E. coli cells (DH5α or TOP10)
5. **Select** on antibiotic plates (e.g., kanamycin for pUA66)
6. **Screen** colonies by colony PCR or diagnostic restriction digest
7. **Verify** by Sanger sequencing

This is the most accessible approach and works well with the ODIN home lab kit, with the caveat that you need to purchase restriction enzymes separately.

##### Option B: Gibson Assembly (Modern, Recommended)

1. Design **20–25 bp homology arms** on each end of the synthetic fragment that overlap with the linearized vector
2. **Linearize** the vector by PCR or restriction digest
3. **Mix** fragment + linearized vector with **Gibson Assembly Master Mix** (NEB, ~\$180 for 10 reactions)
4. **Incubate** at 50°C for 15–60 minutes
5. **Transform** into competent E. coli
6. **Select** and **verify** as above

Gibson assembly is more reliable for multi-fragment assemblies and avoids restriction site constraints, but requires a thermocycler (or water bath at 50°C).

##### Option C: Golden Gate Assembly

1. Design the synthetic fragments with **BsaI** or **BpiI** Type IIS restriction sites that generate unique 4-bp overhangs
2. Mix all fragments + destination vector + BsaI + T4 ligase in a single tube
3. Run a **thermocycling protocol** (37°C digestion / 16°C ligation, ~30 cycles)
4. Transform and select

This is ideal for combinatorial assembly (building many motif variant constructs simultaneously) and is highly efficient (>95% correct clones).

#### Step 4: Transformation into the Target Organism

##### E. coli
- Use **chemically competent cells** (CaCl₂ method or commercial cells like NEB 5-alpha)
- Heat shock: 42°C for 30–45 seconds
- Recovery in SOC medium at 37°C for 1 hour
- Plate on selective medium
- This is straightforward and compatible with a home lab setup

##### Yeast (S. cerevisiae)
- Use the **lithium acetate / PEG / single-stranded carrier DNA** method
- Requires selective plates (SC-URA for URA3-based vectors)
- Incubate at 30°C for 2–3 days
- Requires additional reagents and yeast-specific media components

##### Human Cells
- Use **lipofection** (Lipofectamine 3000, ThermoFisher) or **electroporation**
- Requires mammalian cell culture facility:
  - CO₂ incubator (37°C, 5% CO₂)
  - Biosafety cabinet (BSL-2)
  - Cell lines: HEK293T (easy to transfect), HeLa, or K562
  - Complete growth medium (DMEM + 10% FBS + pen/strep)
- Harvest cells 24–48 hours post-transfection

#### Step 5: Measure Reporter Output

This is the critical quantitative step. The measurement approach depends on the reporter:

##### GFP-based reporters (E. coli, Yeast)

| Method | Equipment | Quantitative? | Cost |
|--------|-----------|---------------|------|
| UV transilluminator / blue light | ~\$50–100 | Semi-quantitative (visual) | Low |
| Plate reader (fluorescence mode) | \$3,000–15,000 (used) | Yes — measures fluorescence/OD₆₀₀ | Medium |
| Flow cytometer | \$15,000–50,000 (used) | Yes — single-cell resolution | High |
| Microscopy (fluorescence) | \$500–5,000 | Semi-quantitative (imaging) | Medium |

**Recommended for E. coli:** Plate reader. Grow cultures in 96-well plates, measure GFP fluorescence normalized to OD₆₀₀ at multiple time points. Each construct should have **≥6 biological replicates** (independent colonies).

**Protocol for plate reader measurement:**
1. Pick 6–12 colonies per construct into 200 µL LB + antibiotic in a 96-well plate
2. Grow at 37°C with shaking for 4–6 hours (mid-log phase, OD₆₀₀ ≈ 0.4–0.8)
3. Read GFP fluorescence (excitation 485 nm, emission 528 nm) and OD₆₀₀
4. Compute **normalized fluorescence** = GFP / OD₆₀₀ for each well
5. Compare across constructs

##### Luciferase-based reporters (Human cells)

- Use **Dual-Luciferase Reporter Assay System** (Promega, ~\$300)
- Requires a **luminometer** or plate reader with luminescence mode
- Measure Firefly and Renilla luciferase activities
- Compute ratio (Firefly/Renilla or vice versa) to control for transfection efficiency

#### Step 6: Statistical Analysis

For each predicted interaction with $n$ replicates per construct:

1. Compute mean normalized reporter values: $\bar{y}_{00}, \bar{y}_{10}, \bar{y}_{01}, \bar{y}_{11}$
2. Estimate the interaction: $\hat{\beta}_{12} = \bar{y}_{11} - \bar{y}_{10} - \bar{y}_{01} + \bar{y}_{00}$
3. Test significance using a **two-way ANOVA** with interaction term, or equivalently, fit the linear model:

$$y_{ijk} = \beta_0 + \beta_1 x_{1,i} + \beta_2 x_{2,j} + \beta_{12} x_{1,i} x_{2,j} + \varepsilon_{ijk}$$

4. Report the $p$-value for $\beta_{12} = 0$
5. Compare the **sign** of $\hat{\beta}_{12}$ to the computational prediction

---

## 4. Feasibility Assessment by Organism

### 4.1 E. coli (Promoter Interactions)

**Feasibility: HIGH** ✅

E. coli is the most tractable system:
- Fast growth (doubling time ~20 min)
- Efficient transformation (10⁶–10⁸ CFU/µg)
- Well-characterized reporter systems
- No specialized equipment beyond basic microbiology

**Timeline:** 2–3 weeks per set of constructs (1 week cloning + verification, 1 week measurements, 1 week analysis and repeats)

**Estimated cost per interaction (4 constructs):**

| Item | Cost |
|------|------|
| Synthetic oligos (4 × gBlocks) | \$200–400 |
| Reporter vector (Addgene) | \$75 (one-time) |
| Restriction enzymes + ligase | \$200–400 (one-time) |
| Competent cells | \$100–200 |
| Sequencing verification (4 samples) | \$20–40 |
| Media, plates, antibiotics | \$50–100 |
| **Total per interaction** | **~\$350–700** |
| **Total for 5 interactions** | **~\$1,500–3,000** |

**Additional equipment needed beyond ODIN kit:**

| Item | Est. Cost | Necessity |
|------|-----------|-----------|
| Plate reader (used BioTek Synergy) | $2,000–5,000 | Essential for quantitative data |
| Thermocycler (for colony PCR / Gibson) | $500–2,000 (used) | Highly recommended |
| Micropipettes (P20, P200, P1000) | \$300–600 | Essential |
| Vortex mixer | \$100–200 | Useful |
| Microcentrifuge | \$200–500 | Essential |

### 4.2 Yeast (UTR Interactions)

**Feasibility: MODERATE** ⚠️

Additional requirements beyond E. coli:
- Lithium acetate transformation protocol
- Selective media (synthetic complete dropout)
- Longer incubation (2–3 days for colonies vs. overnight)
- 30°C incubator (vs. 37°C)

**Additional cost:** ~\$500–1,000 for yeast-specific reagents and media

### 4.3 Human UTR / mRNA Degradation

**Feasibility: LOW** ❌

Requires:
- BSL-2 cell culture facility
- CO₂ incubator (~\$3,000–8,000 used)
- Biosafety cabinet (~\$2,000–5,000 used)
- Cell lines, media, FBS, transfection reagents
- Luminometer or plate reader with luminescence

**Additional cost:** \$10,000–25,000 in equipment alone

**Recommendation:** Collaborate with an established mammalian cell biology lab.

---

## 5. Comparison: Standard Cloning vs. This Experiment

The procedure is fundamentally the same as [standard molecular cloning](https://kchu25.github.io/biology/blog/general/cloning/). Here is a side-by-side comparison:

| Step | Standard Cloning | This Experiment |
|------|-----------------|-----------------|
| **Goal** | Clone a gene of interest into an expression vector | Clone a *regulatory sequence variant* into a reporter vector |
| **Insert source** | PCR from genomic DNA or cDNA | Synthetic oligos (ordered from IDT/Twist) |
| **Vector** | Expression vector (e.g., pET28a) | Reporter vector (e.g., pUA66-GFP) |
| **Assembly** | Restriction/ligation or Gibson | Same — restriction/ligation or Gibson |
| **Transformation** | Into cloning strain (DH5α), then expression strain (BL21) | Into cloning strain, then directly measure |
| **Verification** | Colony PCR + sequencing | Same |
| **Readout** | Protein expression (SDS-PAGE, Western blot) | Reporter fluorescence/luminescence (plate reader) |
| **Key difference** | One construct | **Multiple constructs** (combinatorial motif variants) |

The main additional complexity is **combinatorial**: for each interaction, you need 4 constructs (pairs) or 8 constructs (triplets), each requiring independent cloning and verification. This is why **Golden Gate or Gibson assembly** is preferred — they are more amenable to parallel, high-throughput construct generation.

---

## 6. Advanced: MPRA (Massively Parallel Reporter Assay)

For validating **many interactions simultaneously** (e.g., all significant pairs from the paper), the gold standard is an **MPRA**:

1. **Synthesize** a library of ~1,000–10,000 regulatory sequence variants on an oligonucleotide array (Twist Bioscience, ~\$0.05–0.10/oligo)
2. **Clone** the entire library into a reporter vector in a single reaction
3. **Transform** into cells (E. coli, yeast, or mammalian)
4. **Measure** reporter output for all variants simultaneously using **RNA-seq** (read barcodes associated with each variant)
5. **Quantify** expression for each variant and compute interaction coefficients

**Advantages:**
- Tests thousands of interactions in a single experiment
- Controls for position effects, chromosomal context, etc.
- Publication-quality, high-throughput data

**Disadvantages:**
- Requires access to next-generation sequencing (~\$500–2,000 per run)
- More complex library preparation
- Typically requires an established genomics lab

**Estimated cost for an MPRA experiment:**

| Item | Cost |
|------|------|
| Oligo library synthesis (1,000 variants) | \$5,000–10,000 |
| Cloning reagents | \$500–1,000 |
| Cell culture / transformation | \$200–500 |
| RNA extraction + library prep | \$500–1,000 |
| Sequencing (NextSeq / NovaSeq) | \$1,000–3,000 |
| **Total** | **~\$7,000–15,000** |

---

## 7. The ODIN Home Lab Kit: Honest Assessment

### What the kit provides

- Gel electrophoresis setup
- Incubator (37°C)
- Inoculation loops, plates, basic consumables
- LB media components
- Competent E. coli cells
- Basic transformation protocol

### What's missing for this experiment

| Gap | Why it matters | Solution |
|-----|---------------|----------|
| **No plate reader** | Cannot quantitatively measure GFP fluorescence | Buy used (\$2K–5K) or use a collaborator's |
| **No thermocycler** | Cannot do colony PCR or Gibson assembly | Buy used (\$500–2K) or use water bath |
| **No micropipettes** | Cannot accurately measure µL volumes | Buy a set (\$300–600) |
| **No restriction enzymes** | Cannot cut DNA for cloning | Order from NEB (~\$50–80 each) |
| **No ligase** | Cannot join DNA fragments | Order T4 ligase from NEB (~\$60) |
| **No sequencing** | Cannot verify constructs | Send samples to Plasmidsaurus (\$15/sample) or Genewiz |
| **Limited sterile technique** | Risk of contamination | Use a candle jar or buy a small laminar flow hood |

### Bottom Line

The ODIN kit is a **starting point for learning**, not a research-grade setup. To produce publication-quality data for even the E. coli experiments, you would need an additional **~\$3,000–7,000** in equipment and reagents. The most cost-effective path is:

1. **Minimal viable setup** (~\$3,500 additional): Used plate reader + micropipettes + enzymes + sequencing service → sufficient for E. coli reporter assays
2. **Collaboration** (\$0 additional): Partner with a wet lab that has the infrastructure → can validate across all organisms
3. **Core facility access**: Many universities offer plate reader and sequencing access for ~\$20–50/hour

---

## 8. Recommended Validation Plan

### Phase 1: E. coli proof-of-concept (Home lab feasible with upgrades)
- Select **3 strongest pairwise interactions** from E. coli results
- Build 12 reporter constructs (4 per interaction)
- Measure GFP/OD₆₀₀ with plate reader
- Timeline: 4–6 weeks
- Budget: ~\$1,500–2,500 (excluding equipment)

### Phase 2: Yeast validation (Requires lab access)
- Select **3 strongest UTR interactions** from yeast results
- Use yEGFP reporter in pRS416
- Measure by flow cytometry or plate reader
- Timeline: 6–8 weeks
- Budget: ~\$2,000–3,500

### Phase 3: Human UTR / mRNA degradation (Requires collaborator)
- Select **5 strongest interactions** across human UTR and mRNA degradation datasets
- Use dual-luciferase reporters (psiCHECK-2)
- Measure in HEK293T cells
- Include time-course for mRNA degradation predictions (1h, 4h, 10h)
- Timeline: 8–12 weeks
- Budget: ~\$3,000–5,000 (excluding equipment)

### Phase 4: MPRA (If results are promising)
- Design library covering **all significant interactions** from the paper
- Single experiment validates hundreds of predictions
- Timeline: 3–4 months
- Budget: ~\$10,000–15,000