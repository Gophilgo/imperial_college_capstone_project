# Function 8 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1..x8` to maximize black-box score `y`.
- **What made this hard:** 8D search with weak simple correlations; many dimensions looked low-sensitivity while a few acted as key local levers.
- **Practical meaning:** I moved from broad GP-UCB search to structured local BO, then to trust-region BO near a plateau.

## What EI/UCB/TuRBO Means (Simple)

- **UCB:** `mean + beta*uncertainty`; good for broader uncertainty-led exploration.
- **EI:** expected gain over current best; better for controlled local refinement with `xi` as exploration knob.
- **TuRBO-style local BO:** build a trust region around incumbent and optimize EI inside it, instead of searching full 8D space every step.

## Week-by-Week

- **Week 1**
  - **Idea:** Start with GP + UCB (`kappa=2.0`) on full `[0,1]^8`, then manually avoid extreme corners when interpreting the candidate.
  - **Why I tried it:** I needed an initial global pass in 8D before imposing tighter priors.
  - **What I expected:** A reasonable first high-value region and basic feature pattern.
  - **What happened:** Strong baseline (`~9.74365`), early best.

- **Week 2**
  - **Idea:** Keep UCB but test a practical structured variant (low `x1..x4`, moderate/high `x5..x8`) instead of pure corner pushes.
  - **Why I tried it:** Week 1 suggested a stable backbone pattern but uncertainty on which dimensions truly mattered.
  - **What I expected:** Confirm whether the same backbone stayed competitive.
  - **What happened:** High score (`~9.73005`), near-best.

- **Week 3**
  - **Idea:** Switch to EI exploitation and reduce active dimensions: fix `x1..x6` near the template, optimize mostly `x7/x8` with low `xi` (`~0.001`).
  - **Why I tried it:** Week 1-2 suggested `x7/x8` were likely key local levers while others changed less.
  - **What I expected:** Sharper local improvement by reducing effective search dimensionality.
  - **What happened:** Drop (`~9.6549`), showing extreme `x7/x8` choice was harmful.

- **Week 4**
  - **Idea:** Keep constrained EI but move `x7` back above zero and keep `x8` in a moderate/high band instead of extreme 1.0.
  - **Why I tried it:** Week 3 showed that `x7=0` plus extreme `x8` underperformed.
  - **What I expected:** Recovery toward top cluster without leaving the known basin.
  - **What happened:** Rebound (`~9.72465`), still below best.

- **Week 5**
  - **Idea:** Continue constrained EI on `x7/x8` with fixed `x1..x6`, targeting `x7` upward (`~0.21`) and `x8` near `~0.70` (`xi~0.01`).
  - **Why I tried it:** Week 4 rebound supported a non-zero `x7` and non-extreme `x8` ridge.
  - **What I expected:** Small improvement by refining within that band.
  - **What happened:** New max (`~9.74969`), marginal but real gain.

- **Week 6**
  - **Idea:** Keep local EI, widen `x7/x8` search slightly, allow small `x5` perturbation (`+/-0.02`), and raise exploration mildly (`xi~0.02`) with jitter.
  - **Why I tried it:** I wanted nearby alternatives without collapsing onto one point.
  - **What I expected:** Another small gain or clearer local contour.
  - **What happened:** Slightly lower (`~9.72709`), no new max.

- **Week 7**
  - **Idea:** Tighten to explicit local bands: `x5 in [0.58,0.62]`, `x7 in [0.18,0.28]`, `x8 in [0.68,0.74]`, EI `xi=0.02`, plus repulsion on `(x7,x8)`.
  - **Why I tried it:** Week 6 suggested the good region was narrow but stable.
  - **What I expected:** Near-best confirmation and maybe a tiny improvement.
  - **What happened:** Very close score (`~9.74889`), just below best.

- **Week 8**
  - **Idea:** Narrow further around winning center: `x7 in [0.17,0.23]`, `x8 in [0.64,0.72]`, keep `x5` jitter band, EI `xi=0.02`, light repulsion.
  - **Why I tried it:** Plateau-like behavior suggested micro-refinement around `x7~0.19`, `x8~0.68`.
  - **What I expected:** Tiny local uplift if peak was still unresolved.
  - **What happened:** New max (`~9.75045`), very small improvement.

- **Week 9**
  - **Idea:** Expand safe-region exploration and add GP-vs-RF check; use bounds `x1,x2 in [0,0.15]`, `x3 in [0.06,0.20]`, `x4 in [0.20,0.30]`, `x5 in [0.34,0.46]`, `x6 in [0.55,0.70]`, `xi=0.03`.
  - **Why I tried it:** Plateau gains were tiny, so I tested whether GP-only local assumptions were too narrow.
  - **What I expected:** Either better point from broadened region or evidence to return to tighter local policy.
  - **What happened:** Good but lower (`~9.73952`), not a new max.

- **Week 10**
  - **Idea:** Shift to TuRBO-style local trust-region EI around incumbent with tight trust radii (roughly `0.03-0.04` per dim).
  - **Why I tried it:** I wanted principled local search in a plateau without full-space noise.
  - **What I expected:** Non-trivial local candidate while staying in best basin.
  - **What happened:** Lower (`~9.71173`), no improvement.

- **Week 11**
  - **Idea:** Keep TuRBO-EI but relax trust radii (roughly `0.06-0.08`) and increase exploration pressure to test broader local alternatives.
  - **Why I tried it:** Tight trust region did not improve, so I expanded local scope while staying incumbent-centered.
  - **What I expected:** Possible bounce to a nearby better pocket.
  - **What happened:** Same observed result as prior evaluation (`~9.71173`), no new max.

## Small but Important BO Mechanics I Used

- **Dimension reduction by policy (not by model):**
  - In mid-phase, I fixed `x1..x6` and optimized mostly `x7/x8`.
  - Practical goal: reduce effective search dimension once local levers were identified.

- **Constrained local boxes for key variables:**
  - Used explicit `x7/x8` bands and later tightened them as evidence accumulated.
  - Practical goal: micro-refine near the top ridge instead of broad random drift.

- **`xi` tuning with phase changes:**
  - Lower `xi` in tight exploitation phases; higher `xi` when probing outside plateau.
  - Practical goal: control explore/exploit aggressiveness explicitly.

- **Jitter/repulsion to avoid duplicates:**
  - Added on `(x7,x8)` during local EI phases.
  - Practical goal: avoid repeatedly proposing near-identical points.

- **GP vs RF cross-check in plateau phase:**
  - Used RF feature-importance/ranking check before deciding whether to expand region.
  - Practical goal: reduce model-bias risk from GP-only assumptions.

- **TuRBO-style trust-region transition (late phase):**
  - Moved from manual local boxes to trust-region EI around incumbent.
  - Practical goal: systematic local search with controlled neighborhood size.

## Short Final Takeaway

- Function 8 was dominated by **tiny local gains near a high plateau**; progress came from tightening and retuning local constraints rather than broad global moves.
- The strongest pattern was: **find stable backbone -> isolate key levers (`x7/x8`) -> run constrained local EI -> switch to trust-region BO when marginal gains shrink**.
