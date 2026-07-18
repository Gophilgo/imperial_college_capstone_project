# Function 5 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2, x3, x4` to maximize chemical-process yield `y`.
- **What made this easier than others:** The behavior looked close to unimodal with a strong ridge pattern.
- **Key structure discovered:** High yield consistently came from **high `x2*x3*x4`** and relatively low-to-moderate `x1`.

## What EI-based BO Means (Simple)

- **BO idea:** Fit a surrogate model (GP) to observed data and optimize an acquisition function to choose the next point.
- **EI idea:** EI ("expected improvement") scores points by expected gain above current best.
- **Why EI helps:** It balances:
  - **high predicted mean** (exploit),
  - **high uncertainty** (explore).
- **`xi` in simple terms:** Higher `xi` explores more; lower `xi` exploits more.

## What "Random Restarts" Means (Simple)

- **What gets restarted:** not the whole BO process, but the *inner optimization* of the acquisition function (EI or GP-mean objective).
- **How it works in practice:** run the acquisition optimizer multiple times from different initial points, then keep the best final candidate.
- **Why this matters:** acquisition surfaces in 4D are non-convex, so one start can get stuck in a local optimum; multiple starts reduce that risk.
- **Why outcomes can change a lot:** each start can converge to a different local EI peak; picking the best across restarts often finds a better candidate than a single run.
- **In plain terms:** "restarts" are a robustness check against optimizer path dependence, not random trial-and-error of the whole strategy.

## Week-by-Week

- **Week 1**
  - **Idea:** Start with exploitation around the best observed peak (one-variable perturbation mindset).
  - **Why I tried it:** Function was described as unimodal, so local exploitation should be efficient.
  - **What I expected:** Confirm whether we were already near the dominant basin.
  - **What happened:** Strong point (`~1086.36`), confirming the basin.

- **Week 2**
  - **Idea:** Fit GP and maximize predicted mean (pure exploitation, not uncertainty-heavy BO).
  - **Why I tried it:** Early evidence suggested we were close to optimum, so mean-seeking was appropriate.
  - **What I expected:** Significant gain from cleaner model-guided local moves.
  - **What happened:** New max (`~1935.01`), strong jump.

- **Week 3**
  - **Idea:** Add feature analysis and use `x234_prod = x2*x3*x4` as a constraint signal.
  - **Why I tried it:** Correlation analysis showed `x234_prod` was the strongest predictor of high yield.
  - **What I expected:** Constraining search to high-product regions would improve sample efficiency.
  - **What happened:** New max (`~2066.67`), supporting the product-ridge hypothesis.

- **Week 4**
  - **Idea:** Keep GP exploitation and tighten the `x234_prod` floor (soft constrained objective).
  - **Why I tried it:** I wanted BO to stay in the high-yield manifold instead of drifting to lower-product candidates.
  - **What I expected:** Continued upward movement on the same ridge.
  - **What happened:** New max (`~2323.44`), continued improvement.

- **Week 5**
  - **Idea:** Continue constrained GP exploitation with the same ridge logic.
  - **Why I tried it:** The ridge pattern remained stable across consecutive wins.
  - **What I expected:** Another meaningful gain if ridge still had upward slope.
  - **What happened:** New max (`~2748.83`), large gain.

- **Week 6**
  - **Idea:** Keep searching in the region that was already working (`x1` low-to-moderate, `x2/x3/x4` high), and explicitly test candidates where `x2*x3*x4` is slightly higher than before; use mild exploration plus random-restart acquisition optimization to avoid getting stuck on one local EI optimum.
  - **Why I tried it:** I wanted to verify we were not overfitting one optimizer trajectory.
  - **What I expected:** Either continued gains or a clear sign of plateau.
  - **What happened:** New max (`~2981.55`), ridge still climbing.

- **Week 7**
  - **Idea:** Replace broad search with a tight local EI around the incumbent (roughly `x1` in `0.20-0.24`, `x2` in `0.84-0.90`, `x3/x4` near `1.0`), add jitter so the optimizer does not pick the exact incumbent again, and keep a stronger product constraint (`x2*x3*x4 >= ~0.80`).
  - **Why I tried it:** Needed local exploration without repeatedly proposing the exact incumbent.
  - **What I expected:** Nearby better points while staying on the same high-product manifold.
  - **What happened:** New max (`~3136.42`), method worked.

- **Week 8**
  - **Idea:** Continue the same local ridge-climb policy: keep `x3,x4` at/near `1`, increase `x2` gradually, allow small `x1` drift, and select with local EI + jitter under the product floor so each step is both forward and non-duplicate.
  - **Why I tried it:** Consecutive gains suggested we should continue controlled ridge climbing.
  - **What I expected:** Incremental improvement by pushing `x2` higher while keeping `x3,x4` near 1.
  - **What happened:** New max (`~3327.43`), fourth major climb in sequence.

- **Week 9**
  - **Idea:** Tighten the local box to the high-yield corner (about `x1 in [0.22,0.26]`, `x2 in [0.90,0.98]`, `x3,x4 near 1.0`) and require a higher product target (about `x2*x3*x4 >= 0.82`) before ranking by EI.
  - **Why I tried it:** I wanted to stop backward drift on `x2*x3*x4` and force each proposal to be product-forward.
  - **What I expected:** Aggressive move to the high-product corner.
  - **What happened:** New max (`~4228.12`), very large jump.

- **Week 10**
  - **Idea:** Convert the product rule from a fixed floor to a strict forward rule: compute `best_prod = best_x2*best_x3*best_x4`, set `prod_target = best_prod + margin`, and only accept EI candidates with `x2*x3*x4 >= prod_target`.
  - **Why I tried it:** Previous strict forward rule worked, so consistency mattered more than method changes.
  - **What I expected:** Smaller but still positive gain near saturated corner.
  - **What happened:** New max (`~4460.23`), ridge continued upward.

- **Week 11**
  - **Idea:** Simplify decision logic near saturation; treat `x2=x3=x4≈1` as fixed and focus practical tuning on `x1`.
  - **Why I tried it:** At this stage BO complexity added little compared with direct local refinement.
  - **What I expected:** Marginal gain by fine-tuning the remaining active degree of freedom.
  - **What happened:** New max (`~4462.89`), small but positive endgame improvement.

## Small but Important BO Mechanics I Used

- **Product feature discovery (`x234_prod`):**
  - Identified in **Week 3** as strongest signal and used as a steering variable.
  - Practical goal: collapse 4D intuition into a reliable ridge indicator.

- **Soft product-floor constraint (early-mid phase):**
  - Used from **Weeks 3-8** by penalizing candidates with low `x2*x3*x4`.
  - Practical goal: keep optimizer inside high-yield manifold while still allowing local flexibility.

- **Local EI with jitter/diversity:**
  - Used in **Weeks 7-9** to avoid duplicate proposals at the incumbent.
  - Practical goal: gather new information around the best point without leaving the ridge.

- **Hard monotonic forward constraint (late phase):**
  - Used in **Weeks 9-10** as `x2*x3*x4 >= best_prod + margin`.
  - Practical goal: enforce forward progress on the ridge and prevent backward drift.

- **Search-box tightening around incumbent:**
  - Applied repeatedly in local phases to keep candidate generation near successful region.
  - Practical goal: exploit a confirmed unimodal/high-product structure efficiently.

- **Model simplification near saturation:**
  - In **Week 11**, logic became close to 1D tuning in `x1` because `x2,x3,x4` were effectively saturated near 1.
  - Practical goal: reduce unnecessary model complexity in endgame optimization.

## Short Final Takeaway

- Function 5 was a strong example of **structured exploitation**: once the `x234_prod` ridge was identified, constraint-engineered local BO delivered repeated maxima.
- The key was not changing models every week, but **tightening the right constraints over time** (soft floor -> local EI with jitter -> hard monotonic forward rule -> near-1D final tuning).
