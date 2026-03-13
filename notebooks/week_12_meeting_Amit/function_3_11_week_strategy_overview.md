# Function 3 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2, x3` to maximize transformed response `y` (equivalently, reduce side effects in the original framing).
- **What made this hard:** Scores were all close together (flat landscape), so small differences could be noise or very local structure.
- **Practical meaning:** I needed careful local BO, tight movement controls, and occasional model changes when GP assumptions looked weak.

## What EI-based BO Means (Simple)

- **BO idea:** Fit a GP surrogate to observed points, then choose the next experiment with an acquisition function.
- **EI idea:** EI ("expected improvement") estimates how much a point could improve over current best.
- **Why EI helps:** It balances:
  - **high predicted mean** (exploit),
  - **high uncertainty** (explore where upside may exist).
- **`xi` in simple terms:** Higher `xi` explores more; lower `xi` stays local and exploitative.

## What UCB Means (Simple)

- **UCB idea:** Score points as `mean + beta * uncertainty`.
- **Interpretation:** Bigger `beta` means more exploration; smaller `beta` means more exploitation.
- **Difference vs EI:** EI optimizes expected gain over the incumbent; UCB is a simpler uncertainty bonus on top of mean.

## Week-by-Week

- **Week 1**
  - **Idea:** Start with GP + UCB in full 3D space, so the first proposal balanced uncertain exploration with predicted good regions instead of relying on manual coordinate intuition.
  - **Why I tried it:** Early EDA showed no simple linear trend, so I needed a non-linear model with uncertainty-aware search.
  - **What I expected:** A first data-driven move that tests a promising 3D region while still exploring.
  - **What happened:** Strong start (`~ -0.01175`), best so far at that stage.

- **Week 2**
  - **Idea:** Switch to EI to target improvement magnitude, not just optimistic uncertainty, i.e., rank points by expected gain over incumbent rather than pure uncertainty bonus.
  - **Why I tried it:** After week 1, I wanted more focused progress toward beating the incumbent.
  - **What I expected:** Better exploit/explore balance near the current good zone.
  - **What happened:** Weak point (`~ -0.09608`), not a new max.

- **Week 3**
  - **Idea:** Keep EI but test a higher-`x3` exploratory jump while keeping the other coordinates near the prior good zone, to isolate whether `x3` was the missing lever.
  - **Why I tried it:** I wanted to check whether pushing one coordinate could unlock a better basin.
  - **What I expected:** Either discover a higher-response regime or rule it out quickly.
  - **What happened:** Still weak (`~ -0.06167`), no improvement.

- **Week 4**
  - **Idea:** Pivot back to exploitation and reduce EI exploration pressure (`xi` lower), so candidate selection favored local mean quality after two exploratory misses.
  - **Why I tried it:** Two exploratory misses in a row suggested I was leaving the productive region too easily.
  - **What I expected:** Recovery by staying closer to known good ridge structure.
  - **What happened:** Better than weeks 2-3 but still not best (`~ -0.04611`).

- **Week 5**
  - **Idea:** Keep local EI and push harder on exploitation near the mid-range ridge by narrowing search around the best cluster instead of taking broader global jumps.
  - **Why I tried it:** I expected the best path was precision around the incumbent neighborhood, not broad jumps.
  - **What I expected:** A material improvement if the local ridge estimate was correct.
  - **What happened:** New max (`~ -0.00527`), major jump.

- **Week 6**
  - **Idea:** Add lateral exploration around the new incumbent with diversity pressure, i.e., test nearby alternatives with enforced movement so BO mapped local shape rather than rechecking one point.
  - **Why I tried it:** I wanted to map whether nearby alternatives could beat the new best without collapsing to duplicates.
  - **What I expected:** A near-best point and clearer local geometry.
  - **What happened:** Close but not better (`~ -0.00644`), still useful confirmation.

- **Week 7**
  - **Idea:** Add explicit constraints: enforce `x1_next < x1_incumbent` and require a minimum move on the main active coordinates (`|Δx1| >= 0.02`, `|Δx2| >= 0.02`) before accepting an EI candidate.
  - **Why I tried it:** I wanted to avoid over-sampling the incumbent cluster and deliberately test the second-best region that sat in a lower-`x1` band.
  - **What I expected:** Either a new max from the alternate pocket or evidence it was inferior.
  - **What happened:** Miss (`~ -0.01521`), no new max.

- **Week 8**
  - **Idea:** Keep EI local, and implement three concrete changes: use a rougher GP kernel (`Matérn ν=1.5`), increase the GP noise term (`WhiteKernel` level), and widen length-scale upper bounds; then enforce a minimum candidate step of `0.02` so EI cannot pick a near-duplicate point.
  - **Why I tried it:** The surface looked jittery/flat, so I needed a model less over-smooth and moves that were not tiny repeats.
  - **What I expected:** More robust local ranking under noise and better information per evaluation.
  - **What happened:** Moderate point (`~ -0.00734`), below best.

- **Week 9**
  - **Idea:** Try a Random Forest surrogate instead of GP in a very flat regime, using tree-based non-smooth modeling and tree disagreement as a robustness signal.
  - **Why I tried it:** GP looked uninformative on near-flat values; RF does not assume smoothness and can use tree disagreement as uncertainty proxy.
  - **What I expected:** More robust ranking when smooth-kernel assumptions were too restrictive.
  - **What happened:** Still not a max (`~ -0.00786`), but clarified the plateau challenge.

- **Week 10**
  - **Idea:** Return to smooth GP but switch acquisition from UCB to more exploitative EI in the same local box, so the only major change was acquisition policy under matched search constraints.
  - **Why I tried it:** I wanted a cleaner test of acquisition effect while keeping search region fixed.
  - **What I expected:** A small but real gain if local exploitation was the right endgame behavior.
  - **What happened:** New max (`~ -0.00482`) at `(0.273077, 0.565385, 0.408120)`.

- **Week 11**
  - **Idea:** Keep local smooth-GP exploitation with distance-only constraints around the incumbent (`min_dist_obs`, `min_dist_best`, `max_dist_best`) to enforce disciplined local motion without single-feature hard rules.
  - **Why I tried it:** After a new max, I wanted disciplined local search without hand-coded single-axis rules.
  - **What I expected:** Either slight incremental gain or confirmation of a local pocket border/plateau.
  - **What happened:** Worse point (`~ -0.01497`), suggesting pocket-boundary sensitivity or local noise.

## Small but Important BO Mechanics I Used

- **Minimum move / lateral move constraints:**
  - Used in **Weeks 6-8** to avoid repeating almost identical coordinates near the incumbent; in Week 7 this was explicitly on `x1` and `x2` (`|Δx1|`, `|Δx2|` thresholds).
  - Practical goal: each evaluation should add information about local shape, not just reconfirm one point.

- **Incumbent-side region constraint (Week 7):**
  - I explicitly enforced `x1_next < x1_incumbent` to shift the EI search window toward the lower-`x1` secondary pocket.
  - Practical goal: force a causal test of the second-best region instead of letting BO stay in the incumbent-side cluster.

- **Distance-to-observations guardrails (anti-duplicate):**
  - Applied in local phases to keep candidates away from existing evaluated points.
  - Practical goal: reduce wasted budget on near-duplicates in a flat landscape.

- **Search-radius / local-box control:**
  - Used in late weeks (notably **Weeks 10-11**) to keep BO local around the current best.
  - Practical goal: avoid long exploratory jumps after convergence signals.

- **Acquisition tuning (`xi`, `beta`) by phase:**
  - Early/mid phases used more exploration; later phases reduced exploration for exploitative local refinement.
  - Practical goal: match acquisition aggressiveness to learning stage.

- **Model switching (GP <-> RF):**
  - Switched to RF in **Week 9** when GP looked weak on flat/noisy differences, then returned to GP for constrained local exploitation.
  - Practical goal: test whether smoothness assumptions were helping or hurting at that stage.

- **Distance-only constraints near incumbent (Week 11):**
  - Used `min_dist_obs`, `min_dist_best`, `max_dist_best` style rules without hard-coding one coordinate direction.
  - Practical goal: keep movement disciplined and local while staying model-driven.

## Short Final Takeaway

- Function 3 behaved like a **flat, locally structured surface** where ranking noise and micro-moves mattered a lot.
- Best progress came from **tight local exploitation + explicit movement constraints + method adaptation** (UCB -> EI, temporary RF test, then constrained EI near incumbent).
