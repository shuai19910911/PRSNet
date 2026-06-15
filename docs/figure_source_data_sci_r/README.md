# Source-data rules for redesigned R SCI/BIB figures

This directory is created by `scripts/figures/make_sci_bib_figures.R` when R is available.

Each exported figure must have matching source data:

1. Figure-level source data are written as CSV files with names matching the figure and panel, for example `fig3_main_performance.csv` and `fig5_cross_region_all_methods.csv`.
2. Every plotted value must come from an existing project result table under `docs/figure_source_data/` or `docs/source_data/`; the R script must not invent example values.
3. Schematic panels are allowed only when they define workflow or model vocabulary. Performance, robustness, ablation and interpretation panels must be source-data-driven.
4. Each figure is exported to SVG, PDF, TIFF and PNG preview by the same R backend.
5. Method colors are fixed across figures: RiceGeneFormer blue, SNP-MLP orange, LightGBM green, XGBoost purple, negative/boundary findings red/orange.
6. Figure text must avoid unsupported claims such as state-of-the-art, robustness advantage or biological validation.

Expected output directories after successful R execution:

- `docs/figures_sci_r/`: SVG/PDF/TIFF/PNG figure files.
- `docs/figure_source_data_sci_r/`: source-data CSV files and `SCI_R_FIGURE_MANIFEST.csv`.
