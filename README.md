# hyssong-fmripreprocessing

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Denoises an fMRIPrep-preprocessed BOLD run with AFNI and FSL: brain masking, intensity normalization, nuisance regression (24 motion, WM, CSF, global signal, high-pass) with frame censoring, and spatial smoothing.**

Port of Hayoung Song's post-fMRIPrep notebook: masks the BOLD with the fMRIPrep brain mask (3dcalc), scales intensities (fslmaths -inm 10000), builds high-pass and confound regressors (1dBport, 3dDeconvolve), regresses them out while censoring frames flagged by fMRIPrep motion-outlier columns (3dTproject), and smooths with 3dmerge. Use it to prepare task or rest BOLD for functional-connectivity analyses. Inputs must come from the same fMRIPrep run; tr must be set, and fd_thres is only used to report the percentage of high-motion frames.

| | |
|---|---|
| 📥 **Inputs** | `neuro/func/task` · `neuro/mask` · `neuro/regressors` |
| 📤 **Outputs** | `neuro/func/task` |
| 🐳 **Container** | `nipreps/fmriprep@sha256:102db5fe8b0a34298f2eb2fd5962ad99ff0a948d258cbf08736fcc1b845cad9f` |
| ⚙️ **Resources** | 4 GB memory · 1 h walltime |
| 🔗 **Run it** | [doi.org/10.25663/brainlife.app.958](https://doi.org/10.25663/brainlife.app.958) |

#### 📥 Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `bold` | `neuro/func/task` | yes | fMRIPrep-preprocessed BOLD (space-MNI152NLin2009cAsym, desc-preproc) |
| `mask` | `neuro/mask` (func) | yes | Brain mask matching the BOLD input (from fMRIPrep) |
| `confounds` | `neuro/regressors` | yes | fMRIPrep confounds regressors.tsv for the same BOLD run |

#### 📤 Outputs

| Output | Datatype | Description |
|---|---|---|
| `preproc_bold` | `neuro/func/task` | Brain-masked, intensity-normalized, nuisance-regressed (24 motion parameters + WM + CSF + global signal + high-pass), and spatially smoothed BOLD, ready for functional-connectivity analyses. |

<details>
<summary><b>⚙️ Parameters</b> (4)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `tr` | number |  | Repetition time (seconds) of the BOLD acquisition |
| `fd_thres` | number | `0.5` | Framewise displacement threshold (mm) used to flag/censor high-motion frames |
| `fwhm` | number | `5` | Full-width half-max (mm) of the spatial smoothing kernel applied at the end |
| `hpass_cutoff` | number | `0.0078` | High-pass filter cutoff frequency in Hz (default 0.0078 Hz corresponds to a 128s period) |

</details>

<!-- brainlife-app:end -->

Post-fMRIPrep nuisance regression and smoothing (AFNI/FSL), ported from Hayoung Song's fmripreprocessing notebook

## Origin

Ported from the single-subject/task loop in `preprocess_afterfmriprep.ipynb`
(https://github.com/hyssong/fmripreprocessing), by Hayoung Song
(hyssong@uchicago.edu). The original notebook is included here as
`original_preprocess_afterfmriprep.ipynb` for reference; `main.py` runs the
same six processing steps once per brainlife dataset (one BOLD run at a
time) instead of looping over a hardcoded subject/task list.

Runs inside the existing `nipreps/fmriprep:20.2.3` container (also referenced
by the original repo's `fmriprepscript.sbatch`), which already bundles AFNI
and FSL -- no new image was built for this app.

## Inputs

- `bold` (neuro/func/task)
- `mask` (neuro/mask)
- `confounds` (neuro/regressors)

## Outputs

- `preproc_bold` (neuro/func/task) -- Brain-masked, intensity-normalized, nuisance-regressed (24 motion parameters + WM + CSF + global signal + high-pass), and spatially smoothed BOLD, ready for functional-connectivity analyses.

## Usage

Brainlife.io: run via `braise-app-run`/`braise-app-pipeline` (once registered
with `braise-app-create`), or the web UI.

Locally (outside brainlife): copy `config.json.example` to `config.json`,
fill in real file paths, then run `./main` from this directory. `main` pulls
its own container (`singularity exec docker://...`) -- no local install of
the underlying tool needed, only Singularity itself.

Entrypoint: `main.py`

## Authors

- Gabriele Amorosino <g.amorosino@gmail.com>

## License

MIT, see `LICENSE`.
