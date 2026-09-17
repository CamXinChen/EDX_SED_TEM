# Data

Raw STEM-EDX and SED/4D-STEM datasets are not distributed with this
repository. They are large, acquisition-specific files and are excluded by
`.gitignore`.

To adapt the notebooks, place local inputs here or change the paths in each
notebook's parameter block:

- `Au_300kV_145mm_0.5mrad.zspy` — example Au–Pd calibration filename
- `perovskite_sed.zspy` — example experimental SED filename
- `perovskite_edx_spectrum_image.hspy` — example EDS-TEM spectrum-image filename
- `input_sed.zspy` — example input filename for notebook 04

The calibration filename must encode accelerating voltage, nominal camera
length and convergence angle in the form `300kV_145mm_0.5mrad`.

Notebook 04 continues from notebook 02. Its input is the centred, denoised
signal that notebook 02 produces with `subtract_diffraction_background`,
saved here and reopened in notebook 04's separate environment. The signal must
have shape `(scan_y, scan_x, detector_y, detector_x)`, finite intensities
scaled to [0, 1] and a square detector with a side divisible by 8; notebook 04
checks all three on load.
