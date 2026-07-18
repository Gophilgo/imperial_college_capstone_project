# Function 1 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2` to maximize contamination response `y`.
- **What the data suggested:** Most points are near zero, with a narrow high-value area near the diagonal around `~(0.63, 0.63)`.
- **Practical meaning:** This is not a broad hill. It is more like a narrow ridge/peak, so small moves matter a lot.

## What EI-based BO Means (Simple)

- **BO idea:** Build a model of the function (a GP), then choose the next point using an acquisition score.
- **EI idea:** EI = "expected improvement over current best."  
  It gives high score to points that are either:
  - predicted to be good (**high mean**), or
  - uncertain enough that they might beat the best (**high uncertainty**).
- **Why useful:** EI gives a practical balance between exploitation (go where model says good) and exploration (check uncertain places that could be better).
- **`xi` parameter in simple terms:** Bigger `xi` = more adventurous; smaller `xi` = more conservative/local.

## Week-by-Week

- **Week 1**
  - **Idea:** Make a manual probe toward the diagonal/top-right zone (around the early `(0.6, 0.6)` neighborhood) to test whether the first visible geometric signal was real.
  - **Why I tried it:** The map looked proximity-driven, so I expected stronger signal away from sparse edge points.
  - **What I expected:** A clear direction signal (better or worse than baseline).
  - **What happened:** It produced a new max (`~0.0256`) and gave confidence that structure was geometric.

- **Week 2**
  - **Idea:** Add BO with an extra radial/proximity feature around `(0.6, 0.6)`, so candidate ranking used both raw coordinates and "distance to anchor" as a local-structure cue.
  - **Why I tried it:** I expected distance-to-anchor to help the model rank local candidates better than raw coordinates alone.
  - **What I expected:** Better candidate ranking near the promising region and gradual score gains.
  - **What happened:** Not a new max; good modeling idea, but settings were still too exploratory.

- **Week 3**
  - **Idea:** Add a directional constraint (`x2 >= x1`) on top of local BO, i.e., keep EI search in the same basin but only on the side that looked more promising in the prior map.
  - **Why I tried it:** I expected that restricting search to the empirically stronger side would improve sample efficiency.
  - **What I expected:** Fewer wasted candidates and more stable improvement.
  - **What happened:** Still no new max; constraint alone did not solve it.

- **Week 4**
  - **Idea:** Keep EI but reduce exploration (`xi` down) and tighten the local search box around the incumbent, so BO stopped drifting and focused on close, high-confidence moves.
  - **Why I tried it:** I expected misses were due to drifting too far, not wrong broad region.
  - **What I expected:** More reliable local exploitation and recovery.
  - **What happened:** New max (`~0.0848`), confirming tighter local BO was the right move.

- **Week 5**
  - **Idea:** Add a deterministic gradient-step fallback when BO rankings were unstable: fit a local weighted plane on nearby points and step in its uphill direction if acquisition ranking looked too sensitive.
  - **Why I tried it:** I expected a simple local step to be more robust than over-tuning shortlist filters.
  - **What I expected:** More stable step decisions near the incumbent.
  - **What happened:** Not a new max, but very informative: local surrogate gradients could mislead near sharp peaks.

- **Week 6**
  - **Idea:** Relax one directional rule (allow mild `x2 < x1`) instead of removing all constraints, while keeping the search local, to test whether the true ascent path crossed the old boundary.
  - **Why I tried it:** I expected the hard rule might be blocking the actual ascent direction.
  - **What I expected:** Better points if the true local slope crosses the old boundary.
  - **What happened:** New max (`~0.3589`), strong evidence that selective rule relaxation helped.

- **Week 7**
  - **Idea:** Keep small local steps in the same region with step-size/novelty guardrails, so I tested reproducibility without collapsing onto the exact incumbent point.
  - **Why I tried it:** I expected that if Week 6 was real structure (not luck), conservative follow-up should still improve.
  - **What I expected:** Another gain without major method changes.
  - **What happened:** New max (`~0.5158`), supporting the "stay local and disciplined" approach.

- **Week 8**
  - **Idea:** Use PI/mean-driven local BO with anti-clustering guardrails (minimum step + observation buffer), so the optimizer kept moving along the ridge rather than repeatedly sampling one coordinate.
  - **Why I tried it:** I expected repeated local re-sampling near the incumbent to waste evaluations; guardrails force useful movement.
  - **What I expected:** Continue improving while staying close to the good basin.
  - **What happened:** Breakthrough new max (`~1.7583`), biggest gain in the whole run.

- **Week 9**
  - **Idea:** Switch to TuRBO (trust-region BO): build a small box around the incumbent, fit a local GP inside it, and rank local candidates with EI instead of relying on one global surrogate.
  - **Why I tried it:** In simple terms, TuRBO takes a small local box around the current best and runs BO inside it (fit local GP, score points with EI, pick next point, then expand/shrink box based on success). I expected this to model sharp local geometry better than one global GP.
  - **What I expected:** Better endgame precision near the peak.
  - **What happened:** Not a new max (`~0.8362`), still a strong point but below the best.

- **Week 10**
  - **Idea:** Keep TuRBO for another cycle instead of switching immediately, so the trust-region update logic had enough iterations to re-center after an off-target sample.
  - **Why I tried it:** I expected trust-region adaptation needed more than one step to recover from an off-target sample.
  - **What I expected:** Re-centering and return toward top values.
  - **What happened:** Not a new max (`~0.3690`), showing the local ridge was very brittle.

- **Week 11**
  - **Idea:** Combine TuRBO with deterministic local interpolation/gradient refinement. Concretely, I interpolated `y` as a smooth surface over nearby observed `(x1, x2)` points (RBF interpolation), computed its local gradient, and used that direction for a small deterministic step.
  - **Why I tried it:** I expected that at this stage the problem was micro-navigation (tiny directional choices), so a deterministic local view could complement BO.
  - **What I expected:** More precise moves in the high-value neighborhood.
  - **What happened:** Partial recovery (`~0.9569`) but still below the Week 8 best.

## Small but Important BO Mechanics I Used

- **Distance to incumbent (local control):**
  - Used heavily from **Weeks 4-11** by tightening search around the current best region.
  - Practical goal: once the basin was found, avoid expensive long jumps that were likely off-ridge.

- **Minimum distance to existing points (anti-duplicate rule):**
  - Added in local BO phases (especially **Weeks 7-8** and later local refinements) to avoid near-repeats.
  - Practical goal: make each new evaluation add information instead of re-testing almost the same coordinate.

- **Step-size guardrails (micro-move control):**
  - Used in gradient and local BO phases (e.g., controlled step magnitudes rather than unconstrained moves).
  - Practical goal: on a sharp ridge, large steps can collapse performance quickly; small steps reduce that risk.

- **Directional constraints and controlled relaxation:**
  - Started with directional prior rules (e.g., `x2 >= x1`), then relaxed them selectively (**Week 6**) when evidence suggested the rule blocked ascent.
  - Practical goal: encode prior structure early, then soften rules when data contradicts them.

- **Trust-region mechanics (TuRBO phase):**
  - In **Weeks 9-11**, I used local trust-region search (small box around incumbent, local GP fit, EI-style scoring inside that box).
  - Practical goal: model the local geometry more accurately than one global surrogate near a sharp peak.

- **Deterministic fallback / hybridization:**
  - Used gradient-step fallback and later RBF-interpolation gradient steps when acquisition rankings became unstable.
  - Practical goal: keep decisions interpretable and robust when probabilistic ranking alone was too sensitive.

## Short Final Takeaway

- The strongest pattern across 11 weeks: once the good basin was identified, **tight local BO + careful constraints + small controlled steps** worked best.
- The most important strategic change was not "new model every week," but **iterative hypothesis testing**: adjust one assumption, observe result, and keep what actually improves outcomes.
