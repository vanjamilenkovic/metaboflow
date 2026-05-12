# Untargeted Metabolomics Analysis Pipeline

R scripts for the analysis and visualisation of untargeted metabolomics data, developed as part of PhD research. The pipeline takes feature abundance tables and chemical classification data from standard metabolomics tools (MZmine, SIRIUS/CANOPUS, GNPS) and produces publication-quality figures and statistical outputs.

## Overview

The pipeline is structured as a series of numbered R Markdown (`.Rmd`) scripts that should be run in order. Script `00` builds the core data object used by all downstream scripts.

| Script | Analysis |
|--------|----------|
| `00.phyloseq_object.Rmd` | Build phyloseq object from input files |
| `01.PCA.Rmd` | Principal Component Analysis (PCA) |
| `02.ordination.Rmd` | NMDS, PCoA, and RDA ordination |
| `03.permanova.Rmd` | PERMANOVA and pairwise post-hoc tests |
| `03.1.bubble_plot_pairwisePERMANOVA.Rmd` | Bubble plot of pairwise PERMANOVA results |
| `04.plot_selected_features.Rmd` | Abundance plots for user-defined feature subsets |
| `05.heatmaps.Rmd` | Heatmaps for VIP feature subsets |
| `06.composition.Rmd` | Metabolite class composition bar plots |
| `07.log2fold.Rmd` | Differential abundance (limma) and volcano plot |

## Required input files

Three CSV files are needed to build the phyloseq object (script `00`):

**Abundance table** (`*_STATS.csv`)
- Feature abundance table exported from MZmine
- Features in rows, samples in columns
- Must contain a `row.ID` column

**Chemical classification table** (`*_TAX.csv`)
- Chemical classification from SIRIUS, MolNetEnhancer, and/or CANOPUS
- Exported from Cytoscape
- Features in rows, classification columns
- Must contain a `mappingFeatureId` column
- Expected classification columns: `ChemVistaName`, `ClassyFire.class`, `ClassyFire.most.specific.class`

**Sample metadata** (`*_META.csv`)
- Sample metadata in GNPS format (without the `ATTRIBUTE_` prefix)
- Must contain a `filename` column matching the sample column names in the abundance table
- Expected metadata columns used by these scripts: `timepoint`, `treatment`, `infection`, `group_treatment`

Script `07` additionally requires:
- A log-transformed intensity matrix (`*_STATS_log.csv`) for limma analysis
- A corresponding metadata file (`*_META_log.csv`)

Script `03.1` additionally requires:
- A pre-formatted pairwise PERMANOVA results file (`permanova_final_bubble_data.csv`) — see that script for the required column structure.

## Dependencies

Install all required R packages before running the pipeline:

```r
install.packages(c(
  "here", "tibble", "plyr", "dplyr", "tidyr", "tidyverse",
  "ggplot2", "ggpubr", "ggrepel", "patchwork", "scales",
  "RColorBrewer", "hrbrthemes", "gcookbook",
  "vegan", "limma", "pheatmap"
))

# Bioconductor packages
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

BiocManager::install(c("phyloseq", "microbiome", "microbiomeutilities", "microViz"))

# pairwiseAdonis (GitHub)
remotes::install_github("pmartinezarbizu/pairwiseAdonis/pairwiseAdonis")
```

## How to run

1. Place all input CSV files in your project folder (alongside the `.Rmd` files).
2. Open RStudio and set the project root (or use `here::here()` — it auto-detects the project root from the `.Rproj` file or working directory).
3. Knit or source `00.phyloseq_object.Rmd` first.
4. Run scripts `01` through `07` in any order after that (they all depend only on `pseq`/`pseqs` from script `00`).

Output figures are saved to a `plots/` subfolder and statistical results to a `stats/` subfolder. Both are created automatically if they do not exist.

## Adapting the scripts to your own data

All metadata-specific settings are centralised in a single **configuration
block** at the top of `00.phyloseq_object.Rmd`. Edit that block once and the
changes propagate to all downstream scripts automatically — you should not
need to touch column names anywhere else.

The key variables to set are:

| Variable | What to edit |
|----------|-------------|
| `file_abundance`, `file_taxonomy`, `file_metadata` | Your input CSV file names |
| `col_feature_id`, `col_feature_id_tax`, `col_sample_id` | Identifier column names in each input file |
| `col_color` | Metadata column for the primary grouping variable (used for colour in all plots) |
| `col_shape` | Metadata column for the secondary grouping variable (used for shape/facets) |
| `col_stat_vars` | Metadata columns to test as main effects in PERMANOVA |
| `col_exclude_var` / `col_exclude_val` | Variable and value to exclude from some analyses (set `col_exclude_val <- NULL` to include all samples) |
| `formula_full`, `formula_best`, `formula_rda` | PERMANOVA/RDA model formulas — update `formula_best` after inspecting `full_model` results in script 03 |
| `col_tax_level` | Taxonomy column for compound class composition plots |
| `col_chem_name`, `col_tax_fallback` | Taxonomy columns used to label features in scripts 04 and 05 |
| `limma_levels`, `limma_labels`, `limma_contrast` | Group names and the pairwise contrast for the limma analysis in script 07 |

Scripts `04` and `05` each have their own small configuration block at the
top for the feature ID lists and plot-specific settings (e.g. colour palettes,
number of top features to display).

## Citation

If you use these scripts, please cite:

> Milenkovic, V. (2025). Untargeted metabolomics analysis pipeline [R scripts].
> GitHub. https://github.com/your-username/your-repo-name

## License

MIT License — see `LICENSE` file for details.
