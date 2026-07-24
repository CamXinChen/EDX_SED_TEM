# Data

Raw STEM-EDX and SED/4D-STEM datasets are not distributed with this
repository. They are large, acquisition-specific files and are excluded by
`.gitignore`.

To adapt the notebooks, place local inputs here or change the paths in each
notebook's parameter block:

- `Au_300kV_145mm_0.5mrad.zspy` — example Au–Pd calibration filename
- `perovskite_sed.zspy` — example experimental SED filename
- `perovskite_edx_spectrum_image.hspy` — example EDS-TEM spectrum-image filename

The calibration filename must encode accelerating voltage, nominal camera
length and convergence angle in the form `300kV_145mm_0.5mrad`.
