# Mishra IL-10 Monocyte ATAC-seq Bundle

A bulk ATAC-seq dataset from primary human CD14+ monocytes stimulated with LPS or
TNF in the presence or absence of IL-10. The data is the merged peak set and
post-normalized peak × sample count matrix released with
[Mishra et al. 2025 (Nature Immunology)](https://www.nature.com/articles/s41590-025-02137-3),
GEO accession [GSE261244](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE261244).

## Files

```
mishra_il10_atac_takehome/
├── GSE261244_Merged_ATAC_peaks.bed         3-col BED, hg38, 73,341 merged peaks
├── GSE261244_ATAC_normalized_counts.txt    73,341 peaks × 24 samples, log-scale normalized
└── README.md                               this file
```

## GSE261244_Merged_ATAC_peaks.bed — schema

Tab-separated, no header. Standard 3-column BED with half-open intervals.

| column  | type | description |
|---------|------|-------------|
| `chrom` | str  | chromosome (chr1–chr22, chrX, chrY) |
| `start` | int  | 0-based, half-open start position |
| `end`   | int  | exclusive end position |

Notes:
- 73,341 rows. Coordinates are **hg38** (max chr1 = 248,946,419).
- Autosomes + chrX + chrY only. **No chrM.**
- Peaks are the merged union across all 24 samples (paper's "consensus" set).
- Peak widths range 150–4,150 bp (median 532, mean 597).

## GSE261244_ATAC_normalized_counts.txt — schema

Tab-separated. First column is a peak ID of the form `chr:start-end` matching the
BED file 1:1. Remaining 24 columns are sample IDs.

| column  | type  | description |
|---------|-------|-------------|
| `peak_id` (row index) | str   | `chrom:start-end` — joins to `GSE261244_Merged_ATAC_peaks.bed` |
| 24 sample columns     | float | log-scale normalized accessibility |

Sample naming: `<condition>_D<donor>` where `<donor>` ∈ {D1, D2, D3, D4}.

| condition          | meaning |
|--------------------|---------|
| `Rest`             | unstimulated, no IL-10 |
| `IL10`             | IL-10 only |
| `LPS`              | LPS stimulated |
| `IL10.plus.LPS`    | LPS + IL-10 |
| `TNF`              | TNF stimulated |
| `IL10.plus.TNF`    | TNF + IL-10 |

Design: full 6 × 4 factorial. The same 4 donors are profiled in every condition
(donors are paired). **Total 24 samples.**

Normalization: values are log-scale normalized counts (range approximately
−6 to +11, mean ~2.5). The matrix is **not** raw integer counts — methods that
expect raw counts (DESeq2, edgeR, default count-based GLMs) will misbehave
without a back-transform. Methods built for log-scale data (limma with
`trend=TRUE`, paired t-tests, Wilcoxon, rank-based contrasts) are appropriate.

## Provenance

| stream                                           | source                                                                                                          | license                | notes |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|------------------------|-------|
| `GSE261244_Merged_ATAC_peaks.bed`                | [GEO GSE261244 supplementary](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE261244)                     | per GEO terms          | merged ATAC peak set across all 24 samples, released by paper authors |
| `GSE261244_ATAC_normalized_counts.txt`           | [GEO GSE261244 supplementary](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE261244)                     | per GEO terms          | log-scale normalized peak × sample matrix, released by paper authors |

## Experimental context

- Cell type: primary human CD14+ monocytes
- Culture: RPMI-1640 + 10% FBS + pen/strep + L-glutamine + 20 ng/mL M-CSF
- Stimulation: LPS or TNF, ± IL-10
- Assay: bulk ATAC-seq
- Genome assembly: hg38
- Donors: 4 biological donors, paired across all 6 conditions

## What is *not* in this bundle

The candidate will need to fetch the following external references for steps 5–7:

- **hg38 GTF** for peak-to-gene annotation (e.g. GENCODE v45, Ensembl release 110+,
  or `TxDb.Hsapiens.UCSC.hg38.knownGene` from Bioconductor).
- **hg38 reference FASTA** for motif sequence extraction (e.g. UCSC `hg38.fa`,
  GENCODE primary assembly, or `BSgenome.Hsapiens.UCSC.hg38` from Bioconductor).
- **Motif database** (e.g. JASPAR 2024 CORE vertebrate via `JASPAR2024` Bioconductor
  package, `pyjaspar`, or MEME-format flat file from the JASPAR portal).
- **Gene-set collection** for pathway enrichment (e.g. MSigDB Hallmark via the
  `msigdbr` R package, `gseapy`, or enrichR).

Alignment, peak calling, FRiP / TSS enrichment / fragment-size QC, and library
normalization have already been performed by the paper authors and are upstream
of this bundle.

## Methodology notes

- **hg38, not hg19.** Annotating against hg19 (e.g.
  `TxDb.Hsapiens.UCSC.hg19.knownGene` or a GRCh37 GTF) silently shifts gene
  coordinates and quietly corrupts the peak-to-gene step.
- **Donor pairing.** The 4 donors are paired across all 6 conditions. Donor
  should enter every contrast as a blocking factor. Paired tests are usually
  preferable to unpaired contrasts.
- **Asymmetric stimulus power.** The LPS arm carries far more statistical power
  than the TNF arm in this fixture. Demanding per-stimulus co-significance in
  both `IL10.plus.LPS vs LPS` *and* `IL10.plus.TNF vs TNF` collapses the IL-10
  signature to too few peaks for downstream motif/pathway analysis. A more
  defensible move is to derive the IL-10–suppressed set in the LPS context and
  use the TNF contrast as a sign-consistency check.
- **Motif background choice.** A reasonable IL-10–suppressed foreground is
  paired against either (a) all merged peaks, or (b) LPS/TNF-induced peaks that
  are *not* IL-10–suppressed. Option (b) controls for the baseline inflammatory
  motif landscape and gives a sharper IL-10–specific motif signal.
- **Family-vs-TF granularity.** Motif enrichment alone identifies a TF *family*
  (e.g. ISRE / IRF motifs). Adjudicating IRF1 vs IRF5 vs IRF7 requires
  additional evidence (TF expression, ChIP, perturbation) not in this bundle.
