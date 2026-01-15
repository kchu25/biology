@def title = "5' UTR to Protein Expression: How They Measured It"
@def published = "15 January 2026"
@def tags = ["rna", "screening"]

# 5' UTR to Protein Expression: How They Measured It

## The Big Picture

This is indeed a **sequence-to-expression task**! They're mapping:
- **Input**: 50-nucleotide DNA sequences (5' UTRs)
- **Output**: Protein expression levels

Think of it as: *Does this random sequence make lots of protein or barely any?*

---

## The Measurement Pipeline

### 1. **The Setup**
They created ~500,000 random 50-nt sequences, each placed right before the HIS3 gene:

```
[random 50-nt UTR] → ATG → HIS3 gene
```

### 2. **What is "Selection"?**

> **What does "selection" mean in biology?**
>
> Think of it like a race where the track is rigged to favor certain traits:
> - **Natural selection in evolution**: Organisms with beneficial traits survive and reproduce more
> - **Artificial selection in the lab**: Scientists create conditions where only cells with desired traits thrive
>
> **In this experiment:**
> - The "desired trait" = making lots of His3 protein
> - The "rigged track" = media lacking histidine + 3-AT inhibitor
> - Cells with good UTRs → make His3 → survive → multiply
> - Cells with bad UTRs → can't make enough His3 → starve → die or grow slowly
>
> After 20 hours, the population has been "selected" - it's now enriched with the winners!

**Selection = Competitive growth in harsh conditions**

Think of it like a race where only cells making enough His3 protein can survive and multiply:

- **The challenge**: Growth medium with:
  - ❌ No histidine (essential amino acid)
  - ☠️ 3-AT added (inhibits the His3 enzyme)
  
- **The race**: 
  - Good 5' UTR → lots of His3 protein → cell thrives → divides fast
  - Bad 5' UTR → little His3 protein → cell starves → divides slowly or dies
  
- **Duration**: ~6.2 cell doublings (roughly 20 hours)

After selection, the population is enriched with cells carrying "good" UTRs!

### 3. **The Key Measurement: Enrichment Score**

> **Quick Yeast Biology 101**
> 
> - Yeast cells have their own genome (chromosomes) + can carry extra circular DNA called **plasmids**
> - The plasmid they used is **CEN-based** (centromeric) = "low copy number"
> - CEN plasmids behave like mini-chromosomes: typically **1-2 copies per cell**, maintained stably
> - During cell division, each daughter cell gets ~1 copy (like chromosomes segregating)
> - This is different from "high copy" plasmids (2μ-based) which can have 10-40 copies per cell
> 
> **Why 36 hours matters:**
> - During transformation, a cell might initially pick up multiple different plasmid variants
> - But CEN plasmids segregate 1:1 during division (each daughter gets one copy)
> - After many cell divisions (~36 hours of growth), **random segregation** ensures most cells end up with just one plasmid type
> - It's like shuffling cards repeatedly - eventually each deck becomes homogeneous
> 
> **What about cells with NO plasmid?**
> - Yes, some cells lose their plasmid during division (random segregation isn't perfect)
> - **Solution**: The plasmid has a **LEU2** marker (leucine biosynthesis gene)
> - All growth happens in media **lacking leucine** (SD-Leu)
> - Cells without plasmid → can't make leucine → die!
> - This ensures only plasmid-carrying cells survive and are counted
> 
> **The clean mapping:**
> - After 36 hours: 1 cell ↔ 1 plasmid variant ↔ 1 UTR sequence ↔ 1 expression phenotype
> - Before 36 hours: cells might be "mixed" with multiple variants (messy!)

> **How Do They Actually Sequence the Plasmids?**
>
> **The Basic Workflow:**
> 1. **Harvest cells**: Scoop up millions of yeast cells from the culture
> 2. **Extract plasmids**: Break open the cells and isolate just the plasmid DNA (there are commercial kits for this - like a DNA purification filter)
> 3. **PCR amplification**: The UTR region is tiny compared to the whole plasmid. Use PCR to make millions of copies of just the UTR part (like a molecular copy machine)
> 4. **Add sequencing adapters**: Attach special DNA "handles" to the PCR products so the sequencer can grab them
> 5. **Massively parallel sequencing**: Load onto Illumina NextSeq machine
>
> **What the sequencer does:**
> - Sequences millions of DNA fragments simultaneously
> - Each fragment = one UTR variant
> - Output: A giant list of sequences with counts (e.g., "ACGTACGT... appeared 1,247 times")
>
> **Key insight**: They do this TWICE:
> - **Before selection** (after 36h purification) → input counts
> - **After selection** (after 20h competition) → output counts
>
> Then compare: which sequences became more common? Which disappeared?

**What are they actually counting?**

They're counting **plasmids**, not mRNA! Here's the flow:

1. Each yeast cell carries ~1 plasmid with one 5' UTR variant
2. They extract all plasmids from millions of cells
3. They PCR-amplify and sequence the UTR region
4. Count how many times each UTR variant appears

**Before selection (Input)**:
- Transform yeast with plasmid library
- **Wait ~36 hours** to let cells "purify" (see biology note above)
- Sample the library → sequence → count each variant
- "How common is variant $s$ in our starting population?"

**After selection (Output)**:  
- Let cells compete for ~20 hours in **harsh media** (SD-Leu-His + 1.5 mM 3-AT)
  - **SD-Leu**: Still no leucine (maintains plasmid)
  - **-His**: Now also no histidine (the selection pressure!)
  - **+ 3-AT**: Competitive inhibitor of His3 enzyme (makes it even harder)
- Sample surviving cells → extract plasmids → sequence → count
- "How common is variant $s$ now?"

---

The enrichment score compares these two counts:

$$\text{Enrichment}(s) = \log_2\left(\frac{f_{\text{after}}(s)}{f_{\text{before}}(s)}\right)$$

Where:
- $f_{\text{before}}(s)$ = fraction of plasmids with sequence $s$ in **input library**
- $f_{\text{after}}(s)$ = fraction of plasmids with sequence $s$ **after selection**

**Normalization step** (to account for library size):

$$f(s) = \frac{\text{count}(s) + 1}{\sum_{s'} (\text{count}(s') + 1)}$$

The "+1" is a pseudocount to handle sequences with zero reads.

**Why plasmids and not mRNA?**
- More stable and easier to sequence
- Each cell has ~1 plasmid, so plasmid count ≈ cell count
- If a UTR makes lots of protein → that cell thrives → more copies of that plasmid in the final population!

### 4. **What Does the Score Mean?**

| Enrichment Score | Interpretation |
|------------------|----------------|
| Positive (e.g., +2) | Sequence frequency **increased** → good expression |
| Zero | No change → neutral |
| Negative (e.g., -3) | Sequence frequency **decreased** → poor expression |

**Intuition**: If a sequence helps make lots of His3 protein, those cells grow faster and become more common in the population!

---

## Supporting Measurements

### Flow Cytometry Validation
For 12 sequences, they measured fluorescence with YFP reporter:

$$\text{Fluorescence} \propto \text{Protein expression}$$

Correlation with enrichment: $R^2 = 0.61$ ✓

### Individual Growth Rates  
For 13 sequences, they measured:

$$\text{Max doubling rate} = \max\left(\frac{d(\log \text{OD}_{660})}{dt}\right)$$

Correlation with enrichment: $R^2 = 0.84$ ✓

---

## The CNN Model Task

**Input**: One-hot encoded sequence matrix
- Shape: $(4 \times 50)$ 
- Each column is a position, each row is a nucleotide (A, C, G, T)

**Output**: Predicted enrichment score
- Single continuous value: $\hat{y} \in \mathbb{R}$

**Loss function**: Mean squared error

$$\mathcal{L} = \frac{1}{N}\sum_{i=1}^{N}(y_i - \hat{y}_i)^2$$

Where:
- $y_i$ = observed enrichment score
- $\hat{y}_i$ = predicted enrichment score
- $N$ = number of sequences

**Model performance**: $R^2 = 0.62$ on held-out test set

---

## Key Insight

The enrichment score captures **everything** that affects protein accumulation:
- Transcription efficiency
- mRNA stability  
- Translation initiation & rate
- Protein stability

All rolled into one convenient number! 🎯