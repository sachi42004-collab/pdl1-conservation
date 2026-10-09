# Comparative Conservation Analysis of the PD-1/PD-L1 Immune Checkpoint Axis

An exploratory, hands-on comparative genomics project built with Python and Biopython.

---

## Biological Question

How conserved are the genes encoding the immune checkpoint receptor PD-1 (*PDCD1*) and its ligand PD-L1 (*CD274*) across vertebrate species, and do the resulting evolutionary relationships match expectation?

## Why It Matters

The PD-1/PD-L1 axis is the molecular target of checkpoint-inhibitor cancer immunotherapy (e.g. pembrolizumab). Cross-species sequence comparison indicates how strongly each gene has been constrained over evolutionary time. The gene pair was selected on the basis of a prior literature review of pembrolizumab's mechanism of action.

## Data and Species

### Data Source

NCBI RefSeq mRNA records (nucleotide database), retrieved programmatically through the Entrez E-utilities.

### Species Compared

| Species | Role |
|---|---|
| Human (*Homo sapiens*) | Reference |
| Chimpanzee (*Pan troglodytes*) | Closest relative |
| Mouse (*Mus musculus*) | Model organism |
| Rat (*Rattus norvegicus*) | Model organism |
| Chicken (*Gallus gallus*) | Non-mammalian outgroup |

### Outgroup Selection

Zebrafish (*Danio rerio*) was initially considered as the outgroup. PD-1 has only recently been characterised outside tetrapods, and *PDCD1*/*CD274* are not reliably annotated under matching gene symbols in NCBI for zebrafish. Chicken was used instead: evolutionarily distant from mammals and well annotated.

## Pipeline

| Step | Script | Description |
|---|---|---|
| 1 | `step1_entrez_fetch.py` | Searches NCBI for each gene/species pair and saves one FASTA file per record. Where several transcript variants exist, the longest curated (`NM_`) record is selected as the canonical transcript. |
| 2 | `step2_seqio_parse.py` | Parses the FASTA files with `SeqIO` and checks lengths, accessions and descriptions before further use. |
| 3 | `step3_combine_and_align.py` | Merges each gene's five sequences into one multi-FASTA file and aligns them with MUSCLE v3.8.31, called through `subprocess`. |
| 3b | `step3b_check_alignment.py` | Loads the alignment with `AlignIO` and verifies sequence count, uniform alignment length and gap counts. |
| 4 | `step4_phylo_tree.py` | Computes a pairwise distance matrix (identity model) and builds a Neighbor-Joining tree per gene with `Bio.Phylo`. Trees are exported in Newick format. |

Each transformation step is followed by a verification step before the next stage builds on it.

## Results

### Sequence Identity to Human (*PDCD1*)

Derived from the Step 4 distance matrix (full aligned mRNA sequence):

| Species | Identity to human |
|---|---|
| Chimpanzee | ~96.7% |
| Mouse | ~56.3% |
| Rat | ~56.6% |
| Chicken | ~39.5% |

### Tree Topology

Mouse and rat are closest to each other (~86.0% identity), consistent with a rodent clade. The pattern follows known phylogeny: near-identity within primates, moderate divergence in rodents, and greatest divergence in the chicken outgroup. Distance matrices and Newick trees for both genes are written by Step 4 (`trees/`).

## Interpretation

Sequence similarity decreases in step with evolutionary distance, and the resulting tree topology agrees with accepted species relationships. This agreement serves as a sanity check that the retrieval, alignment and distance calculations behave sensibly. The comparison is exploratory (five species) and does not separate coding-region conservation from conservation across UTRs.

## Limitations

### Transcript Variant Selection

An early version took NCBI's top-ranked hit for *CD274*, which was a shorter, non-canonical splice variant (variant 2, about 176 amino acids) rather than the canonical full-length transcript (variant 1, 290 amino acids). The discrepancy was detected by comparing the translated protein length with the documented length. Selection was changed to the longest curated record among all candidates. Longest-record selection is a heuristic and warrants a manual check for any new gene.

### Predicted Records

Chimpanzee and chicken *PDCD1* sequences are computationally predicted (`XM_`) RefSeq records; curated records were not available for these species.

### Full-mRNA Comparison

Identity is calculated over the whole aligned mRNA, including UTRs, which may evolve under different constraints than the coding sequence.

### Scope

A codon-level comparison of synonymous and non-synonymous differences was considered and left out of this version.

## Key Takeaways

- A database's top-ranked search result is not necessarily the biologically canonical record; outputs need validation against known biology.
- Multiple sequence alignment with an external tool makes cross-species comparison possible despite differing lengths and indels.
- Distance-based phylogenetics converts pairwise differences into an interpretable tree, and expected relationships (e.g. chimpanzee closest to human) provide a check on the pipeline.
- Verification after every stage catches errors before they propagate.

## Tools

- Python 3.13, Biopython (`Entrez`, `SeqIO`, `AlignIO`, `Phylo`)
- MUSCLE v3.8.31 (standalone binary placed in the project root)
- NCBI Entrez E-utilities (`esearch`, `esummary`, `efetch`)

## Reproducing the Analysis

1. Install Biopython (`pip install biopython`) and place the `muscle3` binary in the project root.
2. Set `Entrez.email` in `step1_entrez_fetch.py`.
3. Run the scripts in order: Step 1, Step 2, Step 3, Step 3b, Step 4.

Outputs: `sequences/` (FASTA files), `aligned/` (alignments), `trees/` (Newick trees).