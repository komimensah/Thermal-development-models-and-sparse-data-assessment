# Thermal development models and sparse-data assessment

Code and data accompanying *Symbolic Regression for Temperature Development Functions and the Limits of Sparse Data Validation*.

## Software languages

The analysis is implemented in **Python 3.12**. Symbolic regression is called through **PySR 2.7.2**, which runs its evolutionary equation search using the **SymbolicRegression.jl backend in Julia 1.10.10**. Canonical model fitting, validation summaries, numerical diagnostics, sensitivity calculations, figures and spatial comparisons are implemented in Python. The two `.R` files are optional launchers that call the Python scripts; they are not independent R implementations. There is no requirement to write or run a separate `.jl` analysis script.

The Julia expressions in `rerun_pysr.py`, including the custom loss function, are passed to Julia by PySR. The archived environment files record the Julia dependencies and the Python package versions used for the reported analysis.

## Data and scope

The input comprises 40 published-figure duration summaries digitized with WebPlotDigitizer from Florez-Cuadros et al. (2019), [Effects of Diet Quality and Temperature on Stable Fly Development](https://doi.org/10.3390/insects10070207). The extractor reported approximately 1% extraction error. This estimate is distinct from biological sampling uncertainty. The data contain five temperatures for each of eight endpoint–diet groups. See `data/README.md` for variable definitions and the source-outcome limitation.

The analysis includes six canonical families; training-only symbolic searches at three prespecified penalties; leave-one-temperature-out prediction; reference-expression diagnostics; conditional quadratic digitization sensitivity; and climate-scenario model agreement. The six canonical families are quadratic, Brière, Logan, Lactin, Ratkowsky and the four-parameter high-temperature Sharpe–Schoolfield model.

The saved analysis contains 48 canonical full-data fits, 240 canonical held-temperature fits (including two failed Logan folds), and 160 symbolic searches. All 120 symbolic held-temperature predictions can be checked directly from the archived expression strings without Julia. On these data and configurations, high-temperature Sharpe–Schoolfield has lower five-fold RMSE than symbolic regression in all 24 group-by-penalty comparisons. These comparisons do not establish general biological superiority.

## Installation

Create an isolated Python 3.12 environment and install the numerical-analysis dependencies:

```sh
python -m venv .venv
# macOS / Linux
source .venv/bin/activate
python -m pip install -r requirements-analysis.txt
```

On Windows, activate the environment with `.venv\Scripts\activate` instead.

For new symbolic searches, also install the full requirements:

```sh
python -m pip install -r requirements.txt
```

Install Julia 1.10.10 for the recorded runtime. Set `PYTHON_JULIAPKG_EXE` to the Julia executable if it is not discovered automatically. To use the recorded Julia package environment, set `PYTHON_JULIAPKG_PROJECT` to the absolute path of `environment/` and instantiate its `Project.toml` and `Manifest.toml`. Keep the generated Julia depot outside the Git repository. The original run used macOS ARM64; numerical optimization and stochastic search can vary across platforms even with fixed seeds.

## Check the supplied results without refitting

Run commands from the repository root:

```sh
python code/test_input_checks.py
python code/test_digitization_sensitivity.py
python code/input_checks.py
python code/audit_inputs.py
python code/verify_results.py
python code/summarize_pysr.py
```

`audit_inputs.py` and `summarize_pysr.py` regenerate their audit and summary outputs. They do not repeat canonical optimization or symbolic search. Existing saved numerical records are supplied in `results/`.

## Refit canonical models and regenerate diagnostics

```sh
python code/reanalyse.py
python code/make_figures.py
python code/digitization_sensitivity.py
python code/verify_results.py
```

These commands overwrite their corresponding analysis outputs. Canonical parameters are fitted within each training fold, using fixed bounds and 24 deterministic starting points for nonlinear models. A fold failure is retained; partial-fold RMSE is not substituted for a complete five-fold result.

## Run symbolic searches

```sh
python code/rerun_pysr.py --smoke-test
python code/rerun_pysr.py
python code/summarize_pysr.py
```

The smoke test uses synthetic values to check the runtime. The full command schedules the prespecified searches and skips already recorded runs when their configuration matches. To repeat the searches from the beginning, use a separate copy of the repository and move its existing `results/pysr_runs.csv`, `results/pysr_full_curves.csv`, `results/pysr_configuration.json` and `results/pysr_searches/` into a separate archive first. Keep `results/` present. A configuration mismatch raises an error rather than silently combining experiments.

The training objective and final selection criterion use the fixed rate scale 0.05 day⁻¹. Each penalty (0.001, 0.01 and 0.1) is assessed separately. Held-out responses are not used to discover expressions, optimize constants or select candidates. Seed repetitions describe algorithmic variability rather than confidence intervals.

## Spatial model agreement

The cropped PRISM monthly normals and geographic boundaries are included, so the saved models can be projected with:

```sh
python code/project_spatial_models.py
```

See `spatial/README.md` for the climate source, cropping procedure, domain mask and SSIM settings. The maps compare model projections; they do not validate biological prediction against independent field observations.

## Optional R entry points

```sh
Rscript code/main.R /absolute/path/to/python
Rscript code/Spatial_analysis.R /absolute/path/to/python
```

The first runs the canonical-model and verification scripts. The second projects saved fits. Neither launcher runs symbolic regression. R is unnecessary when the Python commands are used directly.

## Files

- `code/`: commented Python analysis scripts and optional R launchers.
- `data/`: duration summaries, the digitized workbook, eight reference expressions and data documentation.
- `results/`: fit parameters, fold predictions, summaries, verification records and symbolic candidate tables.
- `environment/`: recorded Python versions and Julia project, manifest and runtime metadata.
- `spatial/`: climate grids, boundaries, coordinates, source metadata, model projections and agreement measures.
- `figures/`: separate publication figures. Analysis scripts also generate diagnostic figures here.
- `docs/`: software roles, manuscript wording and release checks.

Python bytecode, machine-specific serialized checkpoints, private editorial records and article PDFs are not distributed in this release. Candidate expressions, their complexity and recorded search parameters are retained as portable text tables. Numerical values are unchanged from the saved analysis; selected descriptive metadata and local output paths were cleaned for this release. See `docs/release_changes.md`.

## Attribution and licensing

The software retains the original MIT license in `LICENSE`. This license does not replace the terms governing third-party climate data or publications. Acknowledge Florez-Cuadros et al. (2019) for the biological source, PRISM Group at Oregon State University for climate normals, and the U.S. Census Bureau for boundaries. The PRISM access date for the archived climate grids is 25 October 2026. The source study reports laboratory experiments; Firth provides geographic context for the source colony, rather than the experimental field location.
