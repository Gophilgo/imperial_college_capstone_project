# Imperial College Capstone Project

Capstone project for the **Professional Certificate in Machine Learning and AI**.

This repository documents a multi-week Bayesian Optimization (BO) study across several black-box objective functions. The work focuses on iterative experimentation, model-guided candidate selection, and practical decision-making under limited evaluation budgets.

## Project Goals

- Optimize multiple unknown objective functions with constrained weekly evaluations.
- Compare and refine BO strategies over time (EI, UCB, trust-region methods, ensemble surrogates, PI-based policies).
- Track when weekly proposals produce new maxima and how reliably BO improves performance.
- Present analysis clearly for academic review and reproducibility.

## Repository Structure

- `initial_data/`
  - Baseline input/output arrays for each function.
- `notebooks/`
  - Weekly analysis notebooks by function.
  - Folders are organized by week (e.g., `week_01`, `week_02`, ..., `week_12`).
  - Each notebook captures:
    - evaluated point(s),
    - observed outcomes,
    - model/acquisition adjustments,
    - recommendation rationale for the next evaluation.
- `notebooks/week_12_meeting_Amit/week_12_bo_overview_manual_evaluations.ipynb`
  - Cross-function summary of manually evaluated points.
  - Includes matrix-style view and new-max tracking.

## Methods Used

The project uses a practical BO toolkit, including:

- Gaussian Process surrogates (`Matern`/`RBF` kernels)
- Expected Improvement (EI)
- Probability of Improvement (PI)
- UCB-style ranking
- Trust-region local search (TuRBO-style)
- Ensemble surrogates (e.g., GP + Random Forest) in selected functions
- Distance-based candidate filters to avoid near-duplicate evaluations

Method choices are adapted per function based on observed behavior (plateaus, local sensitivity, noise characteristics, and remaining evaluation budget).

## How to Run

1. Create and activate a Python environment.
2. Install dependencies used in notebooks (typically: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`, `scipy`, `jupyter`).
3. Open Jupyter and run notebooks from the repository root so relative paths to `initial_data/` resolve correctly.

## Deliverable Orientation

This repository is organized for reviewer readability:

- Week-by-week progression is preserved.
- Strategy changes are explicitly justified in markdown cells.
- Final-week notebooks summarize what was learned and what was recommended next.

## Notes

- All optimization tasks are black-box; domain assumptions are kept minimal and evidence-driven.
- Repeated-point checks are used where relevant to infer local noise behavior.
- Recommendations prioritize robust practical gains over purely theoretical exploration.