# Datasheet — Black-Box Optimisation (BBO) Capstone

This datasheet documents the optimisation decisions, learning and reasoning across the black-box optimisation project. Rather than one datasheet per function, this single sheet covers all eight functions, using per-function tables where the answer is function-specific and shared prose where the approach was common.

## Function overview

**1. Which function does this datasheet describe?**
All eight functions (Function 1–8) of the BBO challenge.

**2. What real-world scenario does each function simulate?**
The functions are synthetic black-box objectives supplied by the course. Where the problem description hinted at a domain, Function 1 behaved like a contamination-detection response and Function 2 like a noisy log-likelihood score. The remaining functions were treated as generic black-box objectives with unknown domain, so no domain assumptions were baked in beyond what the data showed.

**3. What is the dimensionality of the input?**

| Function | Input dimensionality |
|---|---|
| Function 1 | 2D |
| Function 2 | 2D |
| Function 3 | 3D |
| Function 4 | 4D |
| Function 5 | 4D |
| Function 6 | 5D |
| Function 7 | 6D |
| Function 8 | 8D |

All inputs are continuous and bounded to the unit hypercube `[0, 1]^d`.

**4. How many initial data points were provided?**

| Function | Initial input shape | Initial output shape |
|---|---|---|
| Function 1 | (10, 2) | (10,) |
| Function 2 | (10, 2) | (10,) |
| Function 3 | (15, 3) | (15,) |
| Function 4 | (30, 4) | (30,) |
| Function 5 | (20, 4) | (20,) |
| Function 6 | (20, 5) | (20,) |
| Function 7 | (30, 6) | (30,) |
| Function 8 | (40, 8) | (40,) |

**5. What does the output represent?**
A single scalar performance score per function (to be maximised). For Function 1 it reads as a contamination response; for Function 2, a log-likelihood-style score. For the others it is an abstract objective value with no stated units.

## Nature of the data

**1. Structure of the initial dataset.**
Each function is provided as an `initial_inputs.npy` array of shape `(n, d)` and an `initial_outputs.npy` array of shape `(n,)`, stored under `initial_data/function_i/`. Sizes range from 10 points (2D functions) to 40 points (8D function).

**2. How the dataset evolves.**
Exactly one new input/output pair was added per function per week over the challenge (roughly 11–13 weeks). Early weeks spread points broadly to learn scale and shape; later weeks concentrated queries in a tight neighbourhood around the incumbent best. So the dataset grows slowly and shifts from space-covering to locally clustered.

**3. Noise / randomness.**
Noise varied by function. Function 2 showed meaningful noise (repeated-region evaluations differed). Function 8 was found to be effectively deterministic: a point that was accidentally re-submitted returned an identical output, which confirmed near-zero observation noise and justified dropping the noise (white-kernel) term and using a more exploitative, mean-focused search.

**4. Unimodal / multimodal / noisy / smooth (per observations).**

| Function | Observed character |
|---|---|
| Function 1 | Smooth but a narrow high-value ridge near the diagonal (~0.63, 0.63); very sensitive to small moves |
| Function 2 | Noisy, apparently multimodal |
| Function 3 | Low, flat signal with local noise near a shallow pocket |
| Function 4 | Moderately smooth; steady directional gradient |
| Function 5 | Smooth and strongly driven by the product x2·x3·x4; near-monotone toward a corner |
| Function 6 | Rugged 5D landscape; sensitive to x2 |
| Function 7 | Structured; low first two dimensions and higher last dimension favoured |
| Function 8 | Effectively deterministic; sparse active dimensions |

Reasoning came from plots, surrogate (GP) posterior variance, ARD length scales and repeated-point checks.

## Your optimisation strategy

**1. Which optimisation method(s) did you use?**
Primarily Bayesian optimisation with a Gaussian Process surrogate and Expected Improvement (EI) and Upper Confidence Bound (UCB) acquisition. Supplemented per function with: Probability of Improvement (PI) for final exploitative calls, trust-region local search (TuRBO-style), a GP + Random Forest ensemble on Functions 6–8, engineered features, hard constraints, and a deterministic local gradient step (linear plane, later RBF interpolation) as a diagnostic on Function 1.

**2. Why these methods for these functions?**
GP-based BO is sample-efficient, which suits a strict one-query-per-week budget. Random Forest was added where the GP looked unreliable on higher-dimensional, rugged functions, using GP-vs-RF disagreement as an extra uncertainty signal. Feature engineering was used where a structural hypothesis was available (e.g. the product feature on Function 5).

**3. Exploration vs exploitation.**
Controlled through acquisition settings: higher `xi`/`beta` early for exploration, lower later for exploitation. In final weeks the search was restricted to a local box around the incumbent, switched to PI, and constrained (e.g. coordinate bands between the best and second-best points) to squeeze out local gains.

**4. Did the strategy change over the weeks? Why?**
Yes. It evolved from broad exploration to focused exploitation, and from a single global GP to per-function tailoring: adding features, constraints, trust regions or an ensemble in response to what each function revealed. The guiding rule was iterative hypothesis testing — change one assumption, observe the result, keep what improves outcomes.

## Data handling and preprocessing

**1. Rescaling / normalisation.**
Inputs already lived in `[0, 1]^d`. Outputs were standardised for the GP (via `normalize_y` / a scaler) so kernel hyperparameters and acquisition scores were well-behaved.

**2. Surrogate models trained.**
Gaussian Process Regressor as the main surrogate; Random Forest Regressor as a cross-check/ensemble on Functions 6–8. Auxiliary fits on Function 1 (a local linear plane and a scipy RBF interpolation for gradient direction) and a least-squares linear fit on Function 5's product feature.

**3. Surrogate preprocessing.**
GP kernels combined a Matern core (mostly ν=2.5, occasionally ν=1.5 for a rougher surface), sometimes an RBF term, a ConstantKernel for scaling, and a WhiteKernel for noise (removed on the deterministic Function 8). Kernel hyperparameters were fitted by marginal-likelihood optimisation with multiple restarts.

**4. Outliers / unusual points.**
No aggressive outlier removal on such small datasets. Anti-duplicate and minimum-distance filters prevented near-repeat evaluations, and repeated-point checks were used deliberately to probe noise (notably confirming Function 8 was noise-free).

## Weekly iteration and learning

**1. How new points changed understanding.**
Each point sharpened the picture of where the signal lived — for example confirming Function 1's narrow diagonal ridge, and that Function 5 was driven by the product of x2, x3 and x4.

**2. Local optima.**
Encountered on the multimodal/rugged functions (e.g. 2, 6). Detected via flat or oscillating improvements, GP posterior behaviour, and points that scored well locally but failed to generalise.

**3. Most informative queries.**
Boundary and structural-hypothesis points (e.g. pushing Function 5 toward the corner where the product is maximised), and constraint-testing points (e.g. relaxing the x2 ≥ x1 rule on Function 1, which unlocked a new max).

**4. If restarting, what would you do differently?**
Set up per-function structure and feature engineering earlier, diagnose noise up front, and run a quick multi-model check early so a poorly-fitting GP would have a fallback (relevant to the flat Function 1).

## Performance and results

**1–2. Best output achieved and the input that produced it.**

| Function | Best y | Best input (where recorded) |
|---|---|---|
| Function 1 | ≈ 1.7583 | achieved in the week-8 breakthrough (coordinates in the week-8 notebook) |
| Function 2 | ≈ 0.7457 | recorded in the weekly notebooks |
| Function 3 | ≈ -0.0048 | recorded in the weekly notebooks |
| Function 4 | ≈ 0.5518 | recorded in the weekly notebooks |
| Function 5 | ≈ 4490.15 | `0.333333, 1.000000, 1.000000, 1.000000` |
| Function 6 | ≈ -0.1801 | recorded in the weekly notebooks |
| Function 7 | ≈ 2.4051 | `0.000000, 0.158889, 0.160000, 0.299167, 0.313333, 0.654444` |
| Function 8 | ≈ 9.7857 | `0.011639, 0.083291, 0.089482, 0.012930, 0.621958, 0.890087, 0.224722, 0.712482` |

Across the tracked weeks, 46 of 88 weekly evaluations set a new maximum (52.3%), with Functions 5 and 7 improving most reliably.

**3. Confidence that this is near the global maximum.**
Mixed and function-dependent. Functions 5, 7 and 8 show consistent late-stage improvement and stable predictions, suggesting we are on or near the dominant peak. Function 1 is a narrow, brittle ridge, so the best is likely local. Given the tiny budget relative to dimensionality (especially 5D–8D), global optimality cannot be claimed.

**4. Alignment with expectations.**
Where a structural expectation existed, results matched (Function 5's product-driven corner, Function 1's narrow ridge). Flatter functions (3, 4, 6) plateaued near modest bests, consistent with weak or rugged signal.

## Ethical, practical and general considerations

**1. Relation to real-world applications.**
The one-query-per-week, expensive-evaluation setting mirrors hyperparameter tuning, materials/chemistry screening, clinical trial design and simulation-based optimisation, where each evaluation is costly and decisions must be made under uncertainty.

**2. Limitations of the synthetic nature.**
The true functions are never revealed, there are no units or domain semantics, and the very small budgets mean luck in early placement affects outcomes. Conclusions about "difficulty" are inferences, not ground truth.

**3. Would the strategy scale?**
The GP + acquisition core scales to genuinely expensive problems and is standard practice there. In higher dimensions it needs dimensionality reduction or active-subspace methods; the manual per-function tailoring used here would need to be more automated.

**4. Risks / pitfalls for a future user.**
Over-trusting a single predicted value on tiny data, ignoring noise (or assuming noise where there is none), and committing to one surrogate without a fallback. Repeated-point checks, visual inspection alongside metrics, and constraint-based guardrails mitigate these.
