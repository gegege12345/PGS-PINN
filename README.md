# PGS-LCPINN: Feature-Aware Surrogate Learning for Variable-Condition Compressible Flows

This repository provides the code and data organization used for PGS-LCPINN, a physics-guided surrogate framework for parameter-dependent compressible-flow solutions. The framework combines physics-guided sampling (PGS), a multiclass gating network, and feature-specific local residual correction to improve the prediction of moving shocks, near-wall thermal layers, walls, and outlet flow features.

The example problem is a two-dimensional convergent-divergent nozzle with variable nozzle pressure ratio (NPR) and area ratio (AR).

## Repository contents

```text
.
├── snapshot_sampling_fps_vs_pgs.py
├── baseline_pinn.py
├── peps_pinn.py
├── pgs_moe_lcpinn.py
├── postprocess_baseline.py
├── postprocess_peps.py
├── postprocess_lcpinn.py
├── pgspinn_design_optimization.py
└── README.md
```

| Script | Purpose |
|---|---|
| `snapshot_sampling_fps_vs_pgs.py` | Compares farthest-point sampling (FPS) with PGS and exports sampling diagnostics. |
| `baseline_pinn.py` | Trains the global PINN baseline. |
| `peps_pinn.py` | Trains the PEPS sampling variant. |
| `pgs_moe_lcpinn.py` | Trains the PGS-LCPINN model with one global network, a five-class gate, and four local experts. |
| `postprocess_baseline.py` | Evaluates the baseline model on unseen test cases. |
| `postprocess_peps.py` | Evaluates the PEPS model on unseen test cases. |
| `postprocess_lcpinn.py` | Evaluates PGS-LCPINN, including gate and local-expert predictions. |
| `pgspinn_design_optimization.py` | Uses a trained LCPINN surrogate for NPR-dependent AR matching. |

## Data layout

Place the database folders at paths specified in the `Config` class of each script. The default file names expected by the programs are listed below.

```text
data_root/
├── processed_2d/
│   ├── input(2D)_define.csv
│   ├── points_2d_*.csv
│   └── ...
├── processed(Test)_2d/
│   ├── Testinput(2D)_define.csv
│   ├── Testpoints_2d_*.csv
│   └── ...
└── processed_2d_fulltest/
    ├── points_2d_*.csv
    └── ...
```

| Folder | Contents | Use |
|---|---|---|
| `processed_2d` | Training and validation case definitions and pointwise CFD fields. | Sampling analysis and model training. |
| `processed(Test)_2d` | Test-case definitions and surrogate input points. | Inference on unseen cases. |
| `processed_2d_fulltest` | CFD reference fields corresponding to the test cases. | Error analysis, feature identification, and result visualization. |

The `split` column in `input(2D)_define.csv` identifies training and validation cases. Test inputs and CFD reference fields are paired by `CASE_SUFFIX` and absolute spatial coordinates. The post-processing scripts do not use cell IDs or node IDs for this alignment.

## Installation

Python 3.10 or newer is recommended. A CUDA-enabled PyTorch installation is recommended for training.

```bash
python -m venv .venv
source .venv/bin/activate
pip install numpy pandas scipy matplotlib openpyxl torch
```

`scipy.spatial.cKDTree` is used for efficient coordinate matching and MLS neighborhood queries. The scripts provide a slower fallback if SciPy is not available.

## Configuration

Each program defines a `Config` class near the beginning of the file. Before running a script, set at least:

- the database paths;
- the output directory;
- the GPU or CPU device;
- the checkpoint and dataset-summary paths for post-processing;
- any case limits, sampling ratios, or optimization settings required for the intended experiment.

Use absolute paths or paths relative to the working directory consistently. Keep the folder names unchanged unless the corresponding `Config` paths are updated.

## Typical workflow

### 1. Check the database

Confirm that the training/validation case table includes `CASE_SUFFIX` and `split`, and that a pointwise field file is available for every listed case. For test evaluation, confirm that each test input file has a matching CFD reference file in `processed_2d_fulltest`.

### 2. Compare sampling strategies

```bash
python snapshot_sampling_fps_vs_pgs.py
```

This script compares FPS and PGS at the configured sampling ratios and writes sampling tables, Tecplot data, and optional XLSX summaries to its output directory.

### 3. Train surrogate models

Run one or more of the following scripts after configuring their paths and hyperparameters:

```bash
python baseline_pinn.py
python peps_pinn.py
python pgs_moe_lcpinn.py
```

Training outputs include checkpoints and `dataset_summary.json`. Keep the checkpoint, the summary file, and the configuration used for each experiment together; the post-processing scripts require them to restore scaling factors and model settings.

### 4. Evaluate the test set

```bash
python postprocess_baseline.py
python postprocess_peps.py
python postprocess_lcpinn.py
```

The scripts reconstruct dimensional physical quantities, align predictions with CFD reference points, compute global and feature-domain errors, and export tables and visualization files.

### 5. Run AR matching

After the LCPINN checkpoint is available, configure and run:

```bash
python pgspinn_design_optimization.py
```

The optimizer evaluates candidate AR values at prescribed NPR conditions and exports matched designs, objective histories, exit profiles, and figures.

## Outputs

Depending on the selected script and configuration, outputs may include:

- PyTorch checkpoints (`.pth`);
- dataset summaries (`dataset_summary.json`);
- CSV and XLSX error tables;
- Tecplot DAT files;
- PNG/PDF figures;
- sampled-point sets and feature masks;
- optimization histories and matched AR designs.

These generated files are not required to run the code but should be archived with the corresponding model checkpoint for reproducibility.

## Reproducibility notes

- Record the Python and package versions, GPU model, random seed, and device.
- Keep the exact train/validation/test split with each experiment.
- Report the sampling ratio, PGS allocation settings, MLS neighborhood size, feature thresholds, model architecture, loss weights, and training schedule.
- Use the same reference scaling values stored in `dataset_summary.json` for inference and post-processing.
- The CFD fields are numerical reference solutions, not exact analytical ground truth.

## Data and software restrictions

The repository may distribute exported mesh and flow-field data, sampled point sets, feature labels, trained weights, and the Python source code. Do not upload or redistribute proprietary solver software, including ANSYS Fluent, ANSYS ICEM, or NIST REFPROP. Use those packages under their respective licenses.

If experimental benchmark data were obtained from third-party publications, cite the original source and verify redistribution rights before including the raw measurements in a public dataset.

## License

Unless stated otherwise, the Python source code is released under the MIT License. CFD data, sampled point sets, feature labels, and trained model weights should be released separately under the Creative Commons Attribution 4.0 International License (CC BY 4.0), subject to institutional and third-party licensing constraints.

## Citation

If you use this repository, please cite the associated manuscript:


Please also cite the archived code and data release once a DOI has been issued.

## Contact

For questions about the code or data release, contact the first author listed in the associated manuscript.
