# RM-MDE: criterion-agnostic memetic differential evolution for multilevel image thresholding

Code, certified optima and result files behind the revised manuscript *A criterion-agnostic memetic differential evolution for
reliable multilevel image thresholding* (Otsu, Kapur and minimum-cross-entropy criteria, 8-bit images, 2-8 thresholds).

> **Status.** Public repository: https://github.com/chalermwutcm/RM-MDE-multilevel-thresholding (cited in the manuscript's Data availability statement). The licence is still to be chosen by the author
> (see `LICENSE_TO_BE_CHOSEN.txt`); until a LICENSE file is added, no reuse rights are granted.

## What the final results are built on

* **Reference optima** are computed by an exact dynamic program (`source_code/dp_exact_solver.py`, `runtime_scaling.py`); they are certified for all
  120 instances (3 criteria x 8 images x n = 4-8) in `data_results/exact_optima_dp.json`. Success = relative gap to that optimum < 1e-8.
  They are never taken from any optimizer's own output.
* **Segment-additive criteria.** Otsu, Kapur and minimum cross entropy (MCET) share one form evaluated from cumulative moments
  (`source_code/criteria.py`). MCET uses the intensity coordinate x = i, i = 1..256 (gray value + 1). The sensitivity to x = i - 1 is in
  `data_results/mcet_offset_validation.*` and `exact_optima_mcet_gray0.json`: the optimum changes in 19 of 56 instances.
* **RM-MDE** = `rand/1/bin` DE (F = 0.5, Cr = 0.9, NP = 20, G = 60) + exact coordinate-wise local search (cap of 4 sweeps) applied with probability
  p_ls = 0.15 to accepted trials (`source_code/gopt.py :: MDE`).
* **Baselines** (all NP = 20, G = 60, no tuning for this problem):
  * PSO: constriction (0.729, c = 2.05), ring topology; DE: `rand/1/bin`, F = 0.5, Cr = 0.9;
  * **SDE: the parameter setting reported by Ali et al. (2014), F = Cr = 0.25 with F fixed** (`gopt.SDE(F=0.25, Cr=0.25, dither=False)`), evaluated under this
    study's common budget, not their NP = 10D / 200-iteration experiment. The first-version configuration (randomized F, F = 0.5, Cr = 0.9) is kept only as a
    sensitivity result (`dither=True`; `data_results/sde_sensitivity.json`).
  * **ISSA: a one-dimensional adaptation** of the improved sparrow search of Wu and Yuan (2022), which was proposed for two-dimensional entropy
    thresholding (`gopt.ISSA`). The first-version implementation had an error in the handling of rejected moves and is **not** used for any result;
    it is kept for audit in `ISSA_legacy/` (see below).
* **Image quality** is reported as PSNR and SSIM (scikit-image). **FSIM is not reported**: the code behind the FSIM values of the first version is not
  available and was never validated against a reference implementation.

## Environment

Python 3.13.7, numpy 2.2.6, scipy 1.16.1, scikit-image 0.25.2, matplotlib 3.10.5 (`requirements.txt`); single-threaded runs on an AMD Ryzen 9 5900HX
(31 GB RAM, Windows 11). Runtimes quoted in the manuscript depend on this machine. The test images are the `scikit-image` ones (camera, coins, moon,
astronaut, coffee, chelsea, clock, immunohistochemistry); the scripts load them through `skimage.data`.

## Layout

```
source_code/        all scripts (flat; they import each other: criteria, gopt, ablation, ...); paths.py sets the data/figure folders
data_results/       result files (JSON/CSV); see data_results/DATA_INDEX.md for which are final, intermediate or superseded
figures/            figures of the manuscript and of the supplementary analyses
supplementary/      Supplementary_baseline_sensitivity.md (full PSO/DE grid)
review_response/    Response_to_Reviewers.md and ISSA_equation_audit_checklist.md
ISSA_legacy/        defective first-version ISSA + the audit that found the error (not part of the pipeline)
```

Scripts write into `data_results/` and `figures/` next to `source_code/`; set the environment variable `RMMDE_ROOT` to use another root.

## How to reproduce (run from the repository root; times are approximate, single thread)

| Result in the manuscript | Script | Output | Time |
|---|---|---|---|
| Exact optima for the 120 instances (check) | `python source_code/runtime_scaling.py V` | prints mismatches against `exact_optima_dp.json` | seconds |
| Table 2, scalability figure, convergence figure, PSNR/SSIM table | `python source_code/rerun_main.py` | `main_rerun_*.json`, `figures/criteria_scalability.png`, `figures/conv_Cameraman.png` | ~5 min |
| 120-instance study: PSO, DE, RM-MDE entries | `python source_code/regen_stats120.py` | `regen_stats120.json` (checks against `stats_*.json`) | ~7 min |
| 120-instance study: corrected ISSA and SDE entries and the Friedman / Wilcoxon summary | `python source_code/issa_rerun.py` then `python source_code/sde_rerun.py` | `issa_rerun_*.json`, `sde_rerun_*.json` | ~3 + ~3 min |
| DP vs RM-MDE runtime (L = 256 and scaling in L) | `python source_code/runtime_scaling.py A`, `... B` | `runtime_A_L256.json`, `runtime_B_scaling.json` | minutes |
| Seed-matched wall-clock comparison with PSO and DE | `python source_code/time_matched.py`, then `make_time_matched_figs.py` | `time_matched_DE_PSO.*`, `time_trace_*`, `figures/fig_tm_*` | ~19 min |
| MCET intensity-offset sensitivity | `python source_code/mcet_offset_validation.py` | `mcet_offset_*`, `exact_optima_mcet_gray0.json` | ~6 min |
| Ablation: donor x local search (A), SDE-donor scale factor (A2), p_ls (B) | `python source_code/ablation.py A`, `A2`, `B` (also `check`) ; `make_ablation_figs.py` | `ablation_*.json/.csv` | ~5, ~3, ~12 min |
| RM-MDE vs multistart local search | `python source_code/multistart.py`, then `make_multistart_figs.py` | `multistart_C*` | ~9 min |
| SDE settings sensitivity | `python source_code/sde_sensitivity.py` | `sde_sensitivity*` | ~7 min |
| PSO / DE parameter robustness | `python source_code/baseline_sensitivity.py` | `baseline_sensitivity*` | ~18 min |
| ISSA Levy-scaling check (eq. 10) | `python source_code/issa_levy_check.py` | `issa_levy_check.json` | ~2 min |
| Image-level statistics (Wilcoxon, Hodges-Lehmann, Holm, Friedman, bootstrap) | `python source_code/stats_image_level.py` | `stats_image_level*.json/.csv` | seconds (needs the files above) |

Dependencies between runs: `stats_image_level.py` reads the time-matched, ablation (A, A2, B) and multistart outputs; `baseline_sensitivity.py` reads
`time_matched_DE_PSO.csv`; `sde_rerun.py` reads `issa_rerun_stats120.json` and `stats_*.json`.

## Checks performed on this package (what was actually run)

* `python source_code/runtime_scaling.py V`: the vectorized DP reproduces all 120 certified optima (values and threshold vectors): 0 mismatches.
* `python source_code/regen_stats120.py` (run in the project folder before packaging): PSO, DE and RM-MDE reproduce the stored 120-instance results
  exactly (120 of 120 instances each, difference 0.0).
* `python source_code/ablation.py check` and `python source_code/time_matched.py check`: the instrumented copies reproduce `gopt` for the same seed.
* Re-running `stats_image_level.py` and `make_time_matched_figs.py` from this package regenerates `stats_image_level_master.csv` and the three time-matched
  figures byte-identically (the JSON differs only in the folder name quoted in its protocol text).
* `python ISSA_legacy/issa_audit.py` reproduces the audit: the legacy ISSA's returned thresholds do not attain its reported fitness in 720 of 720 runs.

The long experiments listed above were run with the same code in the project folder before packaging; they were **not** all re-run from this package.

## Known limitations

* Eight images, used for every criterion; image-level inference has N = 8 (smallest attainable two-sided p = 0.0078).
* Baselines were not tuned; a small robustness grid is in `supplementary/` and does not establish that no better-tuned baseline exists.
* The 120-instance PSO/DE/RM-MDE entries come from the first-version scripts, which are not in this package; `regen_stats120.py` regenerates them
  and confirms they are identical.
* `quality_mcet_n5.json` (first version; contains FSIM and superseded ISSA/SDE values) and the SDE/ISSA entries of `stats_*.json` are superseded;
  they are kept for traceability only (`data_results/DATA_INDEX.md`).

## Legacy ISSA

`ISSA_legacy/issa_legacy.py` is the first-version implementation, **defective and not used for any reported result**. `ISSA_legacy/issa_audit.py` compares it
with a version in which only the acceptance is corrected and with a version written from the equations of the paper (results: `ISSA_legacy/results/`).
Do not import it from the pipeline.
