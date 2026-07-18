# Imperial College Capstone Project

Capstone project for the **Professional Certificate in Machine Learning and AI**.

This repository documents a multi-week Bayesian Optimization (BO) study across several black-box objective functions. The work focuses on iterative experimentation, model-guided candidate selection, and practical decision-making under limited evaluation budgets.

## Non-Technical Summary

This project tackles eight hidden "black-box" functions: mathematical systems whose inner workings are unknown, where the only way to learn is to submit one set of inputs per week and observe the score that comes back. The goal is to find the inputs that produce the highest score, using as few attempts as possible. To do this, I built a model that learns the shape of each function from past results and suggests where to try next, balancing safe refinement of good regions against exploring uncertain ones. Over the weeks the approach shifted from broad exploration to focused exploitation, and several functions reached new best values. The repository records the full week-by-week reasoning, results, and a retrospective review of what worked and why.

## Project Goals

- Optimize multiple unknown objective functions with constrained weekly evaluations.
- Compare and refine BO strategies over time (EI, UCB, trust-region methods, ensemble surrogates, PI-based policies).
- Track when weekly proposals produce new maxima and how reliably BO improves performance.
- Present analysis clearly for academic review and reproducibility.

## Repository Structure

- `data_sheet.md`
  - Datasheet documenting the data, landscape, strategy and results across all eight functions.
- `model_card.md`
  - Model card describing the optimisation pipeline, performance, limitations and trade-offs.
- `initial_data/`
  - Baseline input/output arrays for each function (`function_i/initial_inputs.npy`, `initial_outputs.npy`).
- `notebooks/`
  - Weekly analysis notebooks by function.
  - Folders are organized by week (e.g., `week_01`, `week_02`, ..., `week_13`).
  - Each notebook captures:
    - evaluated point(s),
    - observed outcomes,
    - model/acquisition adjustments,
    - recommendation rationale for the next evaluation.
- `notebooks/week_11.5_retrospective_analysis/`
  - A mid-project retrospective reviewing the full 11-week strategy per function.
  - `week_11.5_function_1_retrospective_analysis.md` ... `week_11.5_function_8_retrospective_analysis.md`: plain-English, week-by-week write-ups covering what I changed, why, whether it produced a new max, and what I learned.
  - `week_11.5_bo_overview_manual_evaluations.ipynb`: cross-function summary of manually evaluated points, with a matrix-style view and new-max tracking.

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

## Retrospective Analysis

The `notebooks/week_11.5_retrospective_analysis/` folder steps back from the weekly cadence and reviews the reasoning across all functions. Each per-function write-up documents the hypothesis behind every change, whether it produced a new maximum the following week, and what it revealed about the underlying landscape (for example, that Function 1 behaves like a narrow ridge rather than a broad hill).

The recurring conclusion across functions is that the biggest gains came not from switching models every week, but from disciplined, iterative hypothesis testing: adjust one assumption at a time, observe the result, and keep only what measurably improves outcomes. Once a promising region was identified, tight local BO combined with careful constraints and small, controlled steps was consistently the most effective approach.

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