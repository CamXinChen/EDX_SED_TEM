# EDX and 4D-STEM Analysis of Facet-Engineered Perovskite Films

This repository contains three compact, expert-facing Jupyter workflows for
analysing electron microscopy data from facet-engineered FAPbI₃ perovskite
solar-cell films:

1. STEM-EDX chemical-composition analysis
2. reciprocal-space calibration of SED/4D-STEM data using an Au–Pd standard
3. SED preprocessing, lightweight K-means segmentation and phase/orientation
   indexing

The notebooks document the analytical workflow developed for:

> **Stabilizing perovskite solar cells via facet-selective molecular
> engineering**, *Joule* **10** (2026), 102315.  
> <https://doi.org/10.1016/j.joule.2025.102315>

The work was conducted in the Electron Microscopy Group, Department of
Materials Science and Metallurgy, University of Cambridge (March 2024–February
2025).

## Scientific context

The study investigates tris(4-formylphenyl)amine (TFPA), a
benzaldehyde-functional additive that preferentially binds to the less stable
(100) facets of FAPbI₃. TFPA increases the barrier to the α-to-δ phase
transition and promotes (111)-oriented growth. This facet-selective molecular
engineering improves the film microstructure and contributes to enhanced
device efficiency and operational stability.

An integrated FIB-SEM, STEM-HAADF, STEM-EDX and 4D-STEM workflow was used to
connect nanoscale morphology, chemical composition and crystallographic
orientation with device-level behaviour:

- **STEM-HAADF** compared grain morphology and film uniformity between
  stabilized and control specimens.
- **STEM-EDX** assessed compositional uniformity and additive enrichment at
  grain boundaries.
- **4D-STEM/SED** resolved long-range cubic (111) ordering in stabilized films
  and contrasted it with the disordered, multi-phase control.

The notebooks in this repository focus on the computational path from
microscopy signals to interpretable composition and orientation maps.

## Analytical pipeline

```text
STEM-EDX spectrum image
        │
        └── background/peak processing
                └── elemental intensity and composition maps

Au–Pd standard SED scan
        │
        └── Bragg-disk detection and distortion correction
                └── calibrated reciprocal-space scale and ellipse parameters
                                 │
                                 ▼
Experimental SED / 4D-STEM scan
        │
        └── calibration, centring, rebinning and background subtraction
                └── analysis-ready diffraction signal
                        │
                        ├── K-means clustering
                        │       └── representative cluster diffraction patterns
                        │
                        └── simulated-template matching
                                └── phase, orientation and zone-axis maps
```

The K-means route included here is a lightweight, accessible segmentation
method for relatively simple datasets. It is separate from **SIGMA**, the more
advanced unsupervised machine-learning pipeline developed for high-throughput
4D-STEM phase/orientation analysis.

## Repository contents

| Notebook | Purpose | Principal output |
|---|---|---|
| `notebooks/01_au_pd_reciprocal_calibration.ipynb` | Calibrate SED reciprocal space from an Au–Pd standard using py4DSTEM | Reciprocal-space pixel scale, diffraction-origin correction and elliptical-distortion parameters |
| `notebooks/02_sed_phase_orientation_mapping.ipynb` | Preprocess perovskite SED data, segment it with K-means and match experimental patterns against simulated crystallographic templates | Cluster-average diffraction patterns, α/δ phase assignments, orientation/IPF maps and zone-axis estimates |
| `notebooks/03_edx_composition.ipynb` | Process STEM-EDX spectrum images and estimate spatial chemical composition | Elemental intensity and composition maps |

All three notebooks are cleaned, output-free public versions of the working
analysis notebooks.

### 1. Au–Pd reciprocal-space calibration

The calibration workflow:

1. loads a standard-sample 4D-STEM dataset through HyperSpy and passes it to a
   py4DSTEM `DataCube`;
2. removes hot pixels and computes mean/maximum diffraction patterns;
3. estimates the probe size and forms an annular virtual dark-field image;
4. detects Bragg disks using a synthetic probe kernel, with optional CUDA/CuPy
   acceleration;
5. measures and fits the diffraction origin across the scan;
6. fits elliptical distortion from an isolated Au–Pd diffraction ring;
7. models the standard as a 50:50 random-substitutional FCC Au–Pd alloy using
   Vegard's law;
8. refines the reciprocal-space pixel size against the reference lattice; and
9. exports the accelerating voltage, convergence angle, nominal camera length,
   reciprocal-space scale and ellipse parameters to JSON.

The cleaned notebook targets Python 3.11 and records its main library versions
at runtime. It automatically uses CPU processing when a working CUDA/CuPy
device is not detected.

### 2. SED phase and orientation mapping

The perovskite workflow combines preprocessing with a deliberately lightweight
unsupervised and physics-informed analysis:

1. loads a HyperSpy/pyxem diffraction signal from `.zspy` or another
   HyperSpy-readable format (instrument-specific Merlin `.mib` conversion is
   intentionally outside the public notebook);
2. assigns real- and reciprocal-space calibration, crops the scan, estimates a
   linear direct-beam-shift plane, centres the diffraction patterns and rebins
   navigation space;
3. applies either Cartesian difference-of-Gaussians or H-dome background
   removal, selected in the parameter block, followed by radial-percentile
   subtraction in polar space;
4. applies scikit-learn K-means to the diffraction signal and exports summed
   cluster patterns for rapid inspection;
5. constructs template banks from cubic α-FAPbI₃ and 2H hexagonal
   δ-FAPbI₃ crystal structures using orix and diffsims;
6. performs pyxem polar template matching against the simulated phase and
   orientation bank; and
7. converts the best matches into phase/orientation maps, IPF colours and local
   zone-axis estimates.

The working notebook uses five K-means clusters (`n_init=25`) and one-degree
orientation sampling for the cubic α phase. These are experiment-specific
defaults, not universal recommendations.

### 3. PCA-denoised STEM-EDX composition mapping

The EDX workflow:

1. loads a HyperSpy-readable EDS-TEM spectrum image, with explicit options for
   selecting or summing detector signals from multi-signal containers;
2. crops the energy axis, rebins navigation and energy dimensions, converts to
   floating point and materializes lazy data after size reduction;
3. applies PCA/SVD with Poisson-noise normalization and reconstructs the
   spectrum image from selected components;
4. defines the experiment-relevant X-ray lines and assigns manual background
   windows by line name;
5. extracts background-corrected line intensities, clips negative fitted
   intensities and applies Cliff–Lorimer quantification;
6. plots elemental atomic-composition maps; and
7. constructs threshold-guarded elemental-ratio maps and an optional line
   profile.

The cleaned notebook retains components 0–5 and the original Spectra
Cliff–Lorimer factor set as visible defaults. Both are
experiment-specific choices that must be checked against the PCA scree
plot/factors/loadings and the relevant detector calibration before reuse.
Ratio-map pixels remain `NaN` where either signal falls below its declared
threshold, avoiding artificial ratios from weak denominators.

## Design principles

These notebooks are released as concise method references for readers already
familiar with TEM, STEM-EDX and scanning diffraction. They are intended to make
the analysis logic inspectable and adaptable, rather than to provide a
step-by-step introduction to the experimental techniques.

The public notebooks are organised around:

1. dependencies and configuration;
2. experiment-specific parameters;
3. data loading and validation;
4. preprocessing or feature extraction;
5. analysis and quality checks;
6. visualisation and export.

Dataset paths, calibration values, thresholds and crystallographic parameters
are kept near the beginning of each notebook so they can be adapted without
searching through the full workflow.

## Data availability

Raw STEM-EDX and 4D-STEM datasets are not included. The experimental files are
large, instrument-specific and unsuitable for a lightweight demonstration
repository.

The notebooks therefore serve as method records. Users wishing to adapt them
must provide their own data and update:

- input paths and dataset keys;
- scan and detector dimensions;
- detector corrections and calibration values;
- EDX elements, X-ray lines and quantification parameters;
- PCA component selection, background windows and calibrated Cliff–Lorimer
  factors;
- the Au–Pd standard composition and reference reflection;
- crystal-structure files for α-FAPbI₃ and δ-FAPbI₃;
- phase definitions and orientation-search parameters;
- output paths and file formats.

Data availability associated with the publication: **to be added**.

## Environment

The notebooks target the current
[HyperSpy](https://github.com/hyperspy/hyperspy) and
[pyxem](https://github.com/pyxem/pyxem) ecosystem:

- **HyperSpy** provides the multidimensional signal and metadata framework.
- **pyxem** provides diffraction-specific signal classes and tools for
  SED/4D-STEM analysis.
- **eXSpy** provides the current HyperSpy extension for EDS/EDX analysis.
- **py4DSTEM** provides Bragg-disk detection and reciprocal-space calibration
  for the Au–Pd standard.
- **scikit-learn**, **diffsims**, **orix** and **diffpy** support clustering,
  diffraction simulation and crystallographic indexing.

Conda or Mamba with the `conda-forge` channel is the recommended installation
route:

```bash
conda create -n edx-sed-tem -c conda-forge \
    hyperspy pyxem exspy py4dstem scikit-learn diffsims orix diffpy.structure \
    h5py jupyterlab
conda activate edx-sed-tem
jupyter lab
```

Alternatively, install the packages into a clean Python environment with pip:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip
python -m pip install \
    hyperspy pyxem exspy py4dstem scikit-learn diffsims orix diffpy.structure \
    h5py jupyterlab
jupyter lab
```

The Au–Pd notebook currently requests CUDA-accelerated Bragg-disk detection.
CuPy is therefore optional for general inspection but required to run that
cell unchanged on an NVIDIA CUDA system. Install the CuPy build appropriate
for the local CUDA toolchain, or disable the notebook's `CUDA` and
`CUDA_batched` options for CPU execution.

The commands above install the latest mutually compatible releases available
from the selected package channel. Once the notebooks have been consolidated
and tested, the exact working versions will be recorded in `environment.yml`
for reproducibility. Users should prefer that file over installing unpinned
latest releases when reproducing this repository.

Official installation guidance:

- [HyperSpy installation](https://hyperspy.org/hyperspy-doc/current/user_guide/install.html)
- [pyxem installation](https://pyxem.readthedocs.io/en/stable/user_guide/installing.html)
- [eXSpy installation](https://hyperspy.org/exspy/user_guide/install.html)
- [py4DSTEM installation](https://py4dstem.readthedocs.io/en/latest/installation.html)

## Scope and limitations

- Parameters are specific to the acquisition geometry and specimen preparation
  used in the associated study.
- File-loading code may require modification for other detector formats or
  metadata conventions.
- Instrument-specific `.mib` conversion is not included; the public SED
  workflow begins from `.zspy` or another HyperSpy-readable diffraction signal.
- EDX quantification depends on the selected background, X-ray lines,
  correction factors and specimen assumptions.
- Diffraction indexing depends on calibration quality, reference structures,
  phase assumptions and matching thresholds.
- The K-means segmentation is intended as a lightweight exploratory tool; its
  clusters should not be interpreted as crystallographic phases without
  physics-based validation.
- The absence of raw data means the complete published analysis cannot be
  reproduced from this repository alone.

## Citation

If this workflow contributes to published work, please cite the associated
paper:

> Yi Pan, Xin Chen, Zeyu Zhang, Zeping Ou, Ke Zhao, Bo Zhang, Kun Chen,
> Nabonswende Aida Nadege Ouedraogo, Zhenhuang Su, Bingchen He, Cheuk Hin Ho,
> Shanshan Chen, Yujie Zheng, Tingming Jiang, Jianqiang Qin, Juan Du, Xingyu
> Gao, Rui Wang, Caterina Ducati, Qiang Liao and Kuan Sun. “Stabilizing
> perovskite solar cells via facet-selective molecular engineering.” *Joule*
> **10** (2026), 102315.
> <https://doi.org/10.1016/j.joule.2025.102315>

The repository also includes a machine-readable [`CITATION.cff`](CITATION.cff)
record. GitHub uses this file to display a **Cite this repository** option.

## License

The notebooks and repository documentation are released under the
[MIT License](LICENSE). Third-party libraries retain their respective
licences.
