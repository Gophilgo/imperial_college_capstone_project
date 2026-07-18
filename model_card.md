# Model Card — BBO Capstone Optimisation Pipeline

See the [example Google model cards](https://modelcards.withgoogle.com/model-reports) for the general format. This card describes the optimisation pipeline used to select the next query point for each of the eight black-box functions, not a single trained predictor.

## Model Description

**Input:**
- The accumulated history of evaluated points for a function: an inputs array of shape `(n, d)` in `[0, 1]^d` and a matching outputs array of shape `(n,)`.
- Dimensionality ranges from 2D (Functions 1–2) to 8D (Function 8).
- For some functions, engineered features derived from the raw inputs (e.g. a radial distance for Function 1, distance to the line x1 = 0.7 for Function 2, and the product x2·x3·x4 for Function 5).

**Output:**
- A single recommended next input vector to evaluate (the acquisition-optimal point under the current model and constraints), plus the surrogate's predicted mean and uncertainty at that point for diagnostics.

**Model Architecture:**
- **Primary surrogate:** Gaussian Process Regressor. Kernel built from a Matern core (mostly ν=2.5, occasionally ν=1.5 for rougher surfaces), optionally an RBF term, a ConstantKernel for scaling, and a WhiteKernel for observation noise (dropped on the deterministic Function 8). Hyperparameters fitted by marginal-likelihood maximisation with multiple restarts; outputs standardised.
- **Secondary surrogate:** Random Forest Regressor, used as a cross-check and in a GP + RF ensemble on the harder higher-dimensional Functions 6, 7 and 8, with GP-vs-RF disagreement used as an extra uncertainty signal.
- **Acquisition functions:** Expected Improvement (EI) and Upper Confidence Bound (UCB) for most of the run, switching to Probability of Improvement (PI) for final exploitative calls. Acquisition is optimised over a candidate grid/local box with multi-start where relevant.
- **Search control:** trust-region local search (TuRBO-style) near the incumbent, hard constraints (e.g. x2 ≥ x1 on Function 1; coordinate bands between best and second-best points late in the run), minimum-distance/anti-duplicate filters, and step-size caps.
- **Auxiliary/diagnostic components:** a deterministic local gradient step on Function 1 (local linear plane, later a scipy RBF interpolation) and a least-squares linear fit on Function 5's product feature.

## Performance

Performance is measured by the best objective value reached per function and by how often a weekly query set a new maximum (new-max rate), evaluated on the actual challenge outputs.

Best values reached (higher is better):

| Function | Dim | Best y |
|---|---|---|
| Function 1 | 2D | ≈ 1.7583 |
| Function 2 | 2D | ≈ 0.7457 |
| Function 3 | 3D | ≈ -0.0048 |
| Function 4 | 4D | ≈ 0.5518 |
| Function 5 | 4D | ≈ 4490.15 |
| Function 6 | 5D | ≈ -0.1801 |
| Function 7 | 6D | ≈ 2.4051 |
| Function 8 | 8D | ≈ 9.7857 |

Across the tracked weeks, 46 of 88 weekly evaluations (52.3%) set a new maximum. Functions 5 and 7 improved most reliably; Function 5 set a new max in every tracked week. Per-week reasoning and a matrix/new-max overview are in `notebooks/` and `notebooks/week_11.5_retrospective_analysis/week_11.5_bo_overview_manual_evaluations.ipynb`.

## Limitations

- **Very small data, one query per week.** With 10–40 initial points plus ~11–13 weekly additions, the surrogate is weakly constrained, especially in 5D–8D, so predictions and global-optimality claims are uncertain.
- **No single model fits all functions.** The GP is strong on smooth functions but a poor fit for narrow ridges (Function 1) and flat/rugged landscapes (Functions 3, 6), where progress plateaued.
- **Predicted values sit below the incumbent.** The exploitative surrogate often predicts near or below the current best around it, so it confirms rather than dramatically extrapolates.
- **Manual, per-function tailoring.** Features and constraints were hand-designed, which does not scale automatically to new problems.
- **Synthetic objective.** The true function, units and domain are unknown, so interpretation is inference-based.

## Trade-offs

- **Exploration vs exploitation.** Early exploration builds surrogate quality but spends scarce budget; late exploitation cashes it in but risks locking onto a local peak. Managed by lowering `xi`/`beta` and tightening the search over time.
- **Breadth vs depth of models.** Committing to a GP backbone gave depth and interpretability but no automatic fallback when it fit badly; a Random Forest was added selectively rather than running a full multi-model contest.
- **Constraints vs freedom.** Hard constraints (e.g. coordinate bands, x2 ≥ x1) improved sample efficiency and avoided known-bad regions, but risk excluding the true optimum if the prior structure is wrong.
- **Noise modelling.** A WhiteKernel guards against noisy observations but wastes capacity when the function is deterministic; it was removed on Function 8 once repeated points confirmed zero noise.
- **Performance issues.** The pipeline is weakest on narrow-ridge and flat/rugged functions and in the highest dimensions, where limited data and smoothing assumptions blunt the surrogate's guidance.
