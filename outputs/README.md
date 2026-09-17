# Generated outputs

Notebook-generated calibration records, cluster-average diffraction patterns
and phase/orientation maps are written below this directory. Generated files
are excluded from version control by default.

Notebook 04 creates a fresh `run_*` directory under `sigma_sed/` for each
execution, so repeated runs never overwrite one another. Each run holds the
model checkpoints written during training, a `figures/` folder of PNG
diagnostics (`input_overview.png`, `latent_cluster_map.png` and one
`cluster_###_diagnostics.png` per occupied cluster) and, when `SAVE_RESULTS`
is enabled, an `export_*` folder of cluster labels, latent means, cluster
counts, mean diffraction patterns and a run-metadata JSON file.
