# Mishra IL-10 ATAC-seq downstream analysis

This repository contains a downstream bulk ATAC-seq analysis of primary human
CD14+ monocytes from Mishra et al. 2025, focused on how IL-10 modulates
LPS- and TNF-driven inflammatory chromatin programs.

The analysis starts from the paper-released merged ATAC peak set and
library-size-normalized, log-scale peak-by-sample matrix. It does not rerun
alignment, peak calling, FRiP, TSS enrichment or fragment-size QC.

## Deliverables

- `mishra_il10_atac_report.Rmd`: source R Markdown report
- `mishra_il10_atac_report.html`: rendered static HTML report
- `mishra_il10_atac_takehome/`: input peak set, normalized matrix and dataset README
- `references/`: external references used by the report

## Analysis overview

The report implements the seven required take-home sections:

1. Peak set and sample matrix characterization
2. Sample QC and replicate structure
3. Contrast design and differential accessibility
4. IL-10 suppression signature
5. Peak-to-gene annotation
6. TF motif enrichment
7. Pathway enrichment and regulatory interpretation

Each section includes an analysis, biological interpretation and a multiple
choice question designed to test dataset-specific reasoning rather than generic
ATAC-seq knowledge.

## Key methods

- Differential accessibility: donor-adjusted limma-trend on normalized log-scale
  accessibility values
- Genome build: hg38
- Peak annotation: GENCODE v45 GTF
- Motif scanning: `BSgenome.Hsapiens.UCSC.hg38` with JASPAR 2024 CORE vertebrate
  motifs
- Pathway enrichment: MSigDB Hallmark v2024.1

The four donors are paired across all six conditions, so donor is included as a
blocking factor in every contrast. Raw-count GLMs such as DESeq2 or edgeR are
not used because the provided matrix is already normalized and log scale.

## Main biological conclusion

IL-10 does not behave like a global chromatin eraser in this dataset. Instead,
it preferentially suppresses a stimulus-induced, IRF/interferon-linked component
of the LPS chromatin response, while much of the broader inflammatory
accessibility landscape remains present. The TNF arm is used as a directional
consistency check rather than a strict co-significance filter, matching the
asymmetric statistical power described in the dataset README.

The motif and pathway layers support the same model: IL-10-sensitive peaks are
enriched for IRF/ISRE-like sequence grammar and annotate near genes enriched for
interferon alpha/gamma response programs, with weaker but present inflammatory
TNF/NF-kB signal. This aligns with Mishra et al.'s model that IL-10 suppresses
IRF activity and the IFN-beta amplification loop rather than simply blocking
canonical NF-kB/AP-1 signaling genome-wide.

## Reproducing the report

From the repository root:

```bash
RSTUDIO_PANDOC=/opt/anaconda3/bin /Library/Frameworks/R.framework/Resources/bin/Rscript -e 'rmarkdown::render("mishra_il10_atac_report.Rmd", output_file = "mishra_il10_atac_report.html", clean = FALSE)'
```

The report expects the input bundle and reference files to be present in the
paths shown above. Package versions are captured in the report's
`sessionInfo()` section.

## Notes

- BED intervals are treated as standard 0-based, half-open intervals for file
  integrity checks.
- Coordinates are converted appropriately when constructing Bioconductor
  `GRanges` objects.
- Nearest-gene annotation is interpreted as hypothesis generation, not proof of
  enhancer target-gene regulation.
