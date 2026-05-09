Epigenetics Engineer Take Home
===

You are provided a bulk ATAC-seq dataset from primary human CD14+ monocytes, generated
in Mishra et al. 2025 (Nature Immunology), in which IL-10 was used to interrogate the
epigenetic suppression of LPS- and TNF-driven inflammatory and interferon response
programs. The data is the merged peak set and the post-normalized peak × sample count
matrix released with the paper (GEO accession GSE261244):

- 6 conditions × 4 donors = 24 samples:
  `Rest`, `IL10`, `LPS`, `IL10.plus.LPS`, `TNF`, `IL10.plus.TNF`
- 73,341 merged ATAC peaks (hg38, autosomes + chrX + chrY, no chrM)
- Library-size–normalized, log-scale count matrix (peak × sample)

The bundle (`mishra_il10_atac_takehome/`) contains:

```
GSE261244_Merged_ATAC_peaks.bed           chrom, start, end (3-col BED, hg38, 73,341 rows)
GSE261244_ATAC_normalized_counts.txt      peak_id (chr:start-end) rows × 24 sample columns,
                                          values are log-scale normalized counts (~ −6 to +11)
README.md                                 provenance, units, condition decoder, sample naming
```

Suggested reading:

- [Paper — Mishra et al. 2025, Nature Immunology](https://www.nature.com/articles/s41590-025-02137-3)
- [Data](https://console.latch.bio/s/20808408466604513)
- [GEO GSE261244](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE261244)
- [ENCODE ATAC-seq standards](https://www.encodeproject.org/atac-seq/)
- [JASPAR 2024](https://jaspar.elixir.no/)
- [MSigDB Hallmark gene sets](https://www.gsea-msigdb.org/gsea/msigdb/human/collections.jsp)

## Summary

After reading the paper, perform an end-to-end downstream ATAC-seq analysis in a
Jupyter or R Markdown notebook that runs through each of the following steps:

1/ Peak Set & Sample Matrix Characterization
2/ Sample QC and Replicate Structure
3/ Contrast Design & Differential Accessibility
4/ IL-10 Suppression Signature
5/ Peak-to-Gene Annotation
6/ TF Motif Enrichment
7/ Pathway Enrichment & Regulatory Interpretation

Note: this dataset has already been peak-called, merged across samples, and
library-normalized by the paper's authors. You will not be running alignment,
peak calling, FRiP, TSS enrichment, or fragment-size QC. The analysis starts at
the count-matrix level and centers on contrast design, annotation, and
regulatory interpretation.

The matched genome assembly is **hg38**. Steps 5 and 6 will require external
references the candidate must fetch and version themselves: a hg38 GTF (e.g.
GENCODE v45) for peak-to-gene annotation, the hg38 reference FASTA for motif
scanning, and a motif database (e.g. JASPAR 2024 CORE vertebrate). Step 7
requires a Hallmark or Reactome gene-set file (e.g. MSigDB Hallmark v2024.1).
The bundle does not include these.

For each step:

- Choose and implement the most appropriate analysis method. Make sure you
  understand the tool and each chosen parameter well, and that the method is
  appropriate for already-normalized, log-scale data (e.g. limma-trend, t-test,
  rank-based methods — not raw-count GLMs like DESeq2/edgeR without back-transformation).
- Construct a multiple choice question that captures the main biological result
  from this step. Focus on 'what you are trying to do' biologically rather than
  implementation details or numerical results from data analysis methods.

Ensure the questions obey the following properties:

- *Antishortcut*: the question requires the agent to interact with the data and
  cannot be answered with general textbook knowledge or by reading the paper's
  abstract.
- *Durability*: the question passes all possible correct scientific paths from
  different method selection / interpretation and fails the incorrect scientific
  paths.

Your deliverable:

- A file (Jupyter or R Markdown notebook) with clearly documented cells for
  each of the steps.
- Each step has an analysis implementation and a multiple choice question in
  markdown extracting the main scientific or regulatory idea of the step.

## Example

The following examples focus on the IL-10 suppression signature.

A good question focuses on the regulatory 'big picture' and captures key
biological results from the data. The questions can only be answered by doing
analysis with this specific dataset:

```
# GOOD

After running paired `IL10.plus.LPS vs LPS` and `IL10.plus.TNF vs TNF`
contrasts (donor as block, FDR < 0.05), which statement best characterizes
how the IL-10 chromatin-modulation program compares across the two stimuli?

A) Quantitatively similar — IL-10 suppresses a comparable number of peaks
   under both stimuli, and the directional pattern is the same
B) Directionally consistent but quantitatively asymmetric — the program is
   detected far more strongly in the LPS context; the TNF-context effect is
   in the same direction but most peaks fall below per-test FDR, so demanding
   co-significance in both contrasts collapses the signature
C) Directionally opposite — IL-10 suppresses accessibility under LPS but
   enhances it under TNF
D) Stimulus-specific with little overlap — IL-10 targets distinct peak sets
   under LPS vs TNF
```

A brittle question focuses on numerical results and might change with slightly
different analysis methods:

```
# BAD

How many peaks pass adjusted p < 0.05 for the IL-10 + LPS vs LPS contrast?

A) 1,247
B) 2,413
C) 3,856
D) 5,102
```

## Deliverable Instructions

Please submit a static HTML report along with the .ipynb or R Markdown notebook
used to generate this report.
