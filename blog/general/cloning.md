@def title = "Molecular Cloning: From Gene to Glowing Bacteria"
@def published = "22 January 2026"
@def tags = ["experimental-biology"]

# Molecular Cloning: From Gene to Glowing Bacteria

This experiment is your introduction to **recombinant DNA technology**—the same foundational technique that powers massively parallel reporter assays (MPRAs), CRISPR delivery systems, and industrial protein production. Think of it as learning to clone *one* gene so you can later understand how scientists clone *hundreds of thousands* simultaneously.

## The Core Pattern: Gene Swapping

You're performing a classic molecular cloning workflow: **take a gene from one source (RFP), insert it into a plasmid (pGLO), and express it in bacteria**. The same logic applies whether you're swapping one gene or testing 500,000 variants in an MPRA.

### Your Mission: GFP → RFP
- **Input**: pGLO plasmid with GFP gene (~5.4 kb total)
- **Gene of interest**: Red fluorescent protein (RFP, ~700 bp)
- **Output**: Bacteria that glow red instead of green under UV light
- **Measurement**: Visual phenotype (red vs green colonies) + antibiotic selection

This is a **single-variant experiment**. Later, you'll learn how MPRAs scale this to test thousands of promoters, UTRs, or regulatory sequences in parallel using sequencing instead of your eyes.

---

## The Pipeline: Input → Instrument → Measurement

Think of molecular cloning as a multi-stage manufacturing process. Each step transforms your DNA, and each transformation is verified before moving forward.

```
┌─────────────────────┐
│  1. Gene            │  ← RFP template DNA
│     Amplification   │
└──────────┬──────────┘
           │ PCR (Thermal cycler)
           │ Adds compatible overhangs
           ▼
┌─────────────────────┐
│  2. Plasmid         │  ← pGLO with GFP
│     Linearization   │
└──────────┬──────────┘
           │ Restriction digest (NheI, EcoRI)
           │ Removes GFP, creates sticky ends
           ▼
┌─────────────────────┐
│  3. Quality Check   │  ← Verify DNA fragments
│     (Gel)           │
└──────────┬──────────┘
           │ Agarose gel electrophoresis
           │ Expected: 700 bp (RFP), 5 kb (plasmid)
           ▼
┌─────────────────────┐
│  4. DNA Assembly    │  ← Join RFP + plasmid
│     (Ligation)      │
└──────────┬──────────┘
           │ Gibson Assembly (50°C)
           │ Creates recombinant pGLO-RFP
           ▼
┌─────────────────────┐
│  5. Transformation  │  ← Get DNA into E. coli
└──────────┬──────────┘
           │ Heat shock (42°C)
           │ Ca²⁺ makes membranes permeable
           ▼
┌─────────────────────┐
│  6. Selection &     │  ← Antibiotic plates
│     Expression      │     + arabinose switch
└──────────┬──────────┘
           │ 37°C overnight incubation
           │ Only transformed bacteria survive
           ▼
┌─────────────────────┐
│  7. Phenotype       │  ← Red vs green fluorescence
│     Observation     │     under UV light
└─────────────────────┘
```

---

## Stage-by-Stage Breakdown

### 1️⃣ **PCR: Amplify RFP Gene** (Day 1)

**What you're doing:** Making millions of copies of the RFP gene with precisely designed primers that add 20-40 bp "overhangs" complementary to your cut plasmid.

**Instruments:**
- Micropipettes (P2, P10, P20, P200)
- Thermal cycler (PCR machine)

**The reaction:**
- Template DNA (RFP source)
- Forward & reverse primers (with overhang sequences)
- DNA polymerase (Taq or high-fidelity)
- dNTPs (A, T, G, C building blocks)

**What happens:** 30 cycles of:
1. **Denature** (94°C): DNA strands separate
2. **Anneal** (58°C): Primers bind to template
3. **Extend** (68°C): Polymerase copies the sequence

**Output:** 2³⁰ = ~1 billion copies of RFP with compatible ends

**Why it matters:** PCR is *the* workhorse of molecular biology. In MPRAs, you'd use this to amplify entire libraries of 100K+ variants, each with unique barcodes.

---

### 2️⃣ **Restriction Digest: Cut pGLO** (Day 1)

**What you're doing:** Using molecular scissors (restriction enzymes) to cut pGLO at two specific sites, removing the GFP gene.

**Instruments:**
- Micropipettes
- Thermal cycler or heat block (37°C)

**The reaction:**
- pGLO plasmid DNA
- NheI-HF enzyme (cuts at GCTAGC)
- EcoRI-HF enzyme (cuts at GAATTC)
- CutSmart buffer (optimal pH and salt)

**What happens:** 
- Two enzymes cut at different sites flanking GFP
- Linear plasmid (~5 kb) + GFP fragment (~700 bp)
- Both pieces have compatible sticky ends matching your RFP overhangs

**Output:** Opened plasmid ready to receive RFP

**Control reaction:** Same setup but *without* enzymes—this should stay circular and won't accept RFP

---

### 3️⃣ **Agarose Gel Electrophoresis: Verify Success** (Day 1 & 2)

**What you're doing:** DNA fragments migrate through a gel matrix based on size. Smaller = faster. This is your quality control checkpoint.

**Instruments:**
- Microwave (melt agarose)
- Gel casting tray + comb
- Electrophoresis tank + power supply (100V)
- UV transilluminator
- Camera

**The science:**
- DNA is negatively charged (phosphate backbone)
- Electric field pulls it toward the positive electrode
- Agarose mesh slows large fragments more than small ones
- Ethidium bromide intercalates into DNA, fluoresces under UV

**What you see:**
- **Gel #1 (digest)**: Two bands—5 kb plasmid + 700 bp GFP
- **Gel #2 (PCR)**: Single band at 700 bp (RFP)
- **Ladder**: Known sizes for comparison

**Why it matters:** This is how you know your enzymes worked. In MPRAs, you'd use gel electrophoresis or bioanalyzers to verify library construction before sequencing.

---

### 4️⃣ **Gibson Assembly: Stitch DNA Together** (Day 2)

**What you're doing:** Joining RFP to the linearized plasmid using overlapping sequences.

**Instruments:**
- Micropipettes
- Thermal cycler (50°C incubation)

**The reaction mix contains 3 enzymes:**
1. **Exonuclease**: Chews back DNA ends to expose overhangs
2. **DNA polymerase**: Fills in gaps
3. **DNA ligase**: Seals the backbone

**What happens:**
- Overhangs on RFP and plasmid are complementary
- They anneal (stick together) at 50°C
- Enzymes fill gaps and seal nicks
- Result: Circular pGLO-RFP plasmid

**Your 4 experimental conditions:**
1. Uncut pGLO, no RFP (negative control—stays as pGLO-GFP)
2. Uncut pGLO + RFP (negative control—can't incorporate RFP)
3. Cut pGLO, no RFP (negative control—plasmid might religate)
4. **Cut pGLO + RFP** (experimental—should create pGLO-RFP)

**Why it matters:** Gibson assembly is incredibly versatile. You can join 3, 4, or even 10 DNA fragments simultaneously—useful for building complex plasmids or CRISPR constructs.

---

### 5️⃣ **Bacterial Transformation: Deliver DNA** (Day 2)

**What you're doing:** Getting your plasmid into *E. coli* cells that don't naturally take up DNA.

**Instruments:**
- Ice bucket
- Heat block (42°C)
- Incubator (37°C)

**The protocol:**
1. **Competent cells + DNA on ice (30 min)**: Ca²⁺ ions neutralize negative charges on DNA and membrane
2. **Heat shock (42°C, 30 sec)**: Thermal stress creates transient pores
3. **Ice recovery (5 min)**: Pores close, trapping DNA inside
4. **LB broth + 37°C (15 min)**: Cells express antibiotic resistance genes

**What happens at the molecular level:**
- Only ~1% of cells actually take up plasmid
- Those that do express beta-lactamase (ampicillin resistance)
- Arabinose acts as a molecular switch for RFP/GFP expression

**Why it matters:** This is how you introduce *any* foreign DNA into bacteria—whether it's one plasmid or a library of 500K variants for an MPRA.

---

### 6️⃣ **Selection & Expression: Amplify Success** (Day 2-3)

**What you're doing:** Using selective pressure to amplify *only* the cells that took up your plasmid.

**Instruments:**
- Agar plates (LB, LB+amp, LB+amp+arabinose)
- 37°C incubator
- UV light source

**The selection scheme:**

| Plate | What Grows | Why |
|-------|------------|-----|
| LB (no antibiotic) | Everything | No selection pressure |
| LB + amp | Only transformed cells | Beta-lactamase destroys ampicillin |
| LB + amp + arabinose | Transformed + glowing | Arabinose activates RFP/GFP promoter |

**The arabinose switch:**
- **Without arabinose**: AraC protein blocks transcription—no color
- **With arabinose**: AraC changes shape, allows RNA polymerase through—RFP/GFP produced

**Expected results:**
- **Reaction 1-3**: Green colonies (control reactions, original pGLO-GFP)
- **Reaction 4**: Red colonies (experimental, pGLO-RFP)

---

### 7️⃣ **Measurement: Phenotypic Readout** (Day 3)

**What you're measuring:** Visual confirmation that RFP replaced GFP.

**Instruments:**
- UV light (or blue light for GFP/RFP excitation)
- Your eyes (or a fluorescence microscope/plate reader for quantification)

**The readout:**
- Green fluorescence = GFP is being expressed
- Red fluorescence = RFP is being expressed
- No fluorescence = No plasmid, or arabinose not present

**Why this matters:** In single-gene cloning, phenotype is visual. In MPRAs, you'd use:
- **Flow cytometry** to bin cells by fluorescence intensity (10K-400K cells/sec)
- **Next-gen sequencing** to count how many times each variant appears in each bin
- **Ratios** to normalize (e.g., log₂[YFP/RFP] or log₂[RNA/DNA])

---

## Key Instruments: The Molecular Biology Toolkit

**DNA Manipulation:**
- **Micropipettes**: Precision liquid handling (0.1-1000 μL)
- **Thermal cycler**: Programmable heating/cooling for PCR, digests, assemblies
- **Microcentrifuge**: Spin down samples, separate phases

**Verification:**
- **Gel electrophoresis**: Separate DNA by size
- **UV transilluminator**: Visualize DNA (via EtBr) or proteins (via fluorophores)

**Cell Culture:**
- **Incubators**: 37°C for bacterial growth
- **Heat block**: Precise temperature control (42°C for heat shock)
- **Autoclave**: Sterilize media and equipment (not used today, but standard)

**If you scaled this up (MPRA-style):**
- **Flow cytometer (FACS)**: Sort cells by fluorescence
- **Illumina sequencer**: Count DNA/RNA reads for each variant
- **Liquid handlers**: Automate pipetting for 96-well or 384-well plates

---

## The Normalization Concept (Preview for MPRAs)

Notice you're running **4 reactions** (cut/uncut × +/- RFP)? This is experimental design 101: **controls let you isolate variables**.

- **Uncut pGLO**: Tests whether cutting is necessary
- **No RFP**: Tests whether assembly is specific to RFP
- **Cut + RFP**: The experimental condition

In MPRAs, this logic scales:
- **log₂(YFP/RFP)**: Cancels out plasmid copy number, cell size
- **log₂(RNA/DNA)**: Cancels out library representation, sequencing depth
- **Enrichment ratios**: Cancels out PCR bias, initial abundance

You're learning to **normalize** by comparing experimental conditions to controls.

---

## Why This Matters: The Bridge to MPRAs

**What you learned today:**
- Clone a single gene into a plasmid
- Verify each step (gel electrophoresis)
- Transform bacteria and select for success
- Measure phenotype (visual fluorescence)

**How this scales to MPRAs:**
- Clone 10K-500K genes/variants into plasmids simultaneously
- Verify library by sequencing
- Transform bacteria/yeast/mammalian cells in bulk
- Measure phenotype by flow cytometry + sequencing (not eyes)

**The fundamental logic is identical:**
1. Build a DNA construct
2. Introduce it into cells
3. Select for success
4. Measure the outcome

The difference is **throughput** (1 vs 500K) and **measurement technology** (visual vs sequencing). The molecular biology is the same.

---

## From Wet Lab to Data

Your final output today is a **qualitative observation**:
```
Reaction     LB+amp+ara result
1 (uncut, no RFP)    → Green colonies
2 (uncut + RFP)      → Green colonies  
3 (cut, no RFP)      → Few/no colonies
4 (cut + RFP)        → Red colonies ✓
```

In an MPRA, you'd get a **quantitative table**:
```
sequence              log₂(activity)
ATCG...TGCA          2.35
GGTA...CCAA          -1.42  
TCCC...TAAG          5.67
```

Same principles, different scale. You're building the foundation.