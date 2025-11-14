@def title = "PDB Format: Visual Guide"
@def published = "14 November 2025"
@def tags = ["proteins"]

# PDB Format: Visual Guide

## Example PDB Lines with Explanations

| PDB Line | What It Means |
|----------|---------------|
| `HEADER    HYDROLASE                               15-MAR-09   3FGQ` | **File header** - Tells you this is a HYDROLASE enzyme, deposited on March 15, 2009, with ID code 3FGQ |
| `TITLE     CRYSTAL STRUCTURE OF HUMAN LYSOZYME` | **Title** - Human-readable description of what structure this file contains |
| `ATOM      1  N   MET A   1      27.340  24.430   2.614  1.00  9.67           N` | **Atom #1** - Nitrogen atom from Methionine (MET), chain A, residue #1, at position (27.340, 24.430, 2.614) Ångströms in 3D space |
| `ATOM      2  CA  MET A   1      26.266  25.413   2.842  1.00  9.48           C` | **Atom #2** - Alpha carbon (CA) of the same Methionine residue, different 3D coordinates |
| `ATOM      3  C   MET A   1      26.913  26.639   3.531  1.00  9.38           C` | **Atom #3** - Backbone carbon of Methionine. Still residue #1 because it's the same amino acid |
| `ATOM    130  N   GLY A   2      28.123  27.018   3.154  1.00  8.12           N` | **New residue** - Now on amino acid #2 (Glycine). The protein chain is building up sequentially |
| `HETATM 1001  O   HOH A 201      31.245  20.183   5.123  1.00 15.23           O` | **Water molecule** - HETATM = non-protein atoms. HOH = H₂O water, numbered separately (201) |
| `CONECT 1001 1002 1003` | **Bond connections** - Tells you atom 1001 is bonded to atoms 1002 and 1003 |
| `TER     131      GLY A   2` | **Chain terminator** - Marks the end of chain A at residue 2 |
| `END` | **File end** - Signals the end of the PDB file |

---

## Understanding ATOM Line Columns

Breaking down: `ATOM      1  N   MET A   1      27.340  24.430   2.614  1.00  9.67           N`

| Position | Content | Meaning |
|----------|---------|---------|
| 1-6 | `ATOM` | Record type (this is an atom) |
| 7-11 | `1` | Atom serial number (unique ID) |
| 13-16 | `N` | Atom name (Nitrogen) |
| 17-20 | `MET` | Residue name (Methionine amino acid) |
| 22 | `A` | Chain identifier |
| 23-26 | `1` | Residue sequence number (position in chain) |
| 31-38 | `27.340` | X coordinate (Ångströms) |
| 39-46 | `24.430` | Y coordinate (Ångströms) |
| 47-54 | `2.614` | Z coordinate (Ångströms) |
| 55-60 | `1.00` | Occupancy (how often atom is in this position) |
| 61-66 | `9.67` | Temperature factor (how much atom vibrates) |
| 77-78 | `N` | Element symbol |

---

## Key Intuitions

**Think of it like a construction manual**
- Each line is an instruction: "Place this specific atom at these exact coordinates"
- Read top to bottom to build the entire molecule

**Hierarchical structure**
- Atoms → Residues (amino acids) → Chains → Complete structure
- Like: Letters → Words → Sentences → Paragraph

**It's just text**
- Despite being 3D structural data, PDB files are plain text
- You can open them in Notepad and read them
- Software visualizes them as beautiful 3D structures

**Coordinates are everything**
- The x, y, z numbers are the "secret sauce"
- They define the exact 3D shape that determines biological function
- Measured in Ångströms (1 Å = 0.0000000001 meters)