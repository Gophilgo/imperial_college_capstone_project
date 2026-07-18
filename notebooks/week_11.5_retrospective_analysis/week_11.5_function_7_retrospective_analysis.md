# Function 7 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1..x6` to maximize ML-model performance score `y`.
- **What made this tricky:** High-dimensional interactions, weak linear signals, and sensitive local structure around a narrow high-performing region.
- **Practical meaning:** I had to combine BO with strong local priors, then progressively relax/tune constraints as evidence accumulated.

## What EI/UCB Means (Simple)

- **UCB idea:** Score points with `mean + beta * uncertainty`; higher `beta` explores more.
- **EI idea:** Score points by expected gain over current best; `xi` controls exploration intensity.
- **How I used both:** UCB for broader uncertainty-led moves early or when testing alternates, EI for tighter local refinement after finding a good basin.

## Week-by-Week

- **Week 1**
  - **Idea:** Start with GP + UCB (`kappa=2.0`) on full `[0,1]^6`, then use ARD lengthscales to identify which coordinates are actually sensitive before adding tighter hand constraints.
  - **Why I tried it:** In 6D, I needed both a first candidate and a feature-sensitivity signal.
  - **What I expected:** A directionally useful first point and rough variable-importance map.
  - **What happened:** Modest positive (`~0.0345`), baseline established.

- **Week 2**
  - **Idea:** Keep UCB but manually reject obvious corner proposals and keep candidates in a practical mid-range regime (especially keep `x6` around `0.5-0.8` and avoid extreme `x3/x4/x5` settings) after Week 1.
  - **Why I tried it:** Week 1 showed naive UCB could drift to unrealistic settings.
  - **What I expected:** Better practical candidate quality and stronger score.
  - **What happened:** Large improvement (`~1.3138`), second-best at that time.

- **Week 3**
  - **Idea:** Switch from UCB to EI with `xi=0.01`, so candidate ranking is based on expected gain over incumbent instead of uncertainty bonus alone after the Week 2 surprise point.
  - **Why I tried it:** I wanted a method that explicitly optimizes improvement over incumbent rather than pure uncertainty bonus.
  - **What I expected:** Better chance of finding a new local ridge.
  - **What happened:** New max (`~1.6455`), major step up.

- **Week 4**
  - **Idea:** Keep EI (`xi=0.01`) but exploit around the discovered ridge: very low `x1/x2/x3`, moderate `x4/x5`, and high `x6` (roughly the Week 3 neighborhood) instead of broad exploration.
  - **Why I tried it:** Week 3 looked like a real basin and needed local confirmation.
  - **What I expected:** Incremental gain from nearby refinement.
  - **What happened:** Drop (`~1.0249`), showing local brittleness (especially low `x4` underperformed).

- **Week 5**
  - **Idea:** Re-center EI around Week 3 structure and target coordinates close to the prior best profile (near-zero `x1/x2/x3`, `x4` around `0.20-0.25`, `x5` around `0.36-0.38`, high `x6`) rather than low-`x4` variants.
  - **Why I tried it:** Week 4 miss suggested the right basin but wrong local direction.
  - **What I expected:** Return toward high-score region.
  - **What happened:** Further drop (`~0.7102`), still below best.

- **Week 6**
  - **Idea:** Raise EI exploration to `xi ~ 0.02-0.03`, enforce a soft `x4` floor (`>= ~0.15`), cap `x6` below `0.75`, and add a repulsion/diversity term on `(x4,x6)` to avoid repeated corner-like proposals.
  - **Why I tried it:** I needed to escape a potentially over-constrained local trajectory without abandoning the basin.
  - **What I expected:** Nearby alternative pocket with better balance.
  - **What happened:** New max (`~1.7805`), strong recovery.

- **Week 7**
  - **Idea:** Keep EI at `xi=0.025` with explicit bounds `x1/x2/x3 in [0,0.12]`, `x4 in [0.20,0.35]`, `x5 in [0.30,0.50]`, `x6 in [0.55,0.75]`, plus repulsion on `(x4,x6)` so candidates spread around the same basin.
  - **Why I tried it:** Week 6 proved the constrained neighborhood was productive; next step was local map refinement.
  - **What I expected:** Small improvement or robust near-best confirmation.
  - **What happened:** Slightly lower (`~1.7220`), still second-best region quality.

- **Week 8**
  - **Idea:** Keep the same basin but tighten around the winning template (`x3` near `0.12`, `x4` around `0.24-0.27`, `x5` around `0.40`, `x6` around `0.61-0.70`), and add GP-vs-RF comparison to check whether the local ranking direction is model-robust.
  - **Why I tried it:** Plateau behavior suggested model-bias risk from GP-only assumptions.
  - **What I expected:** Better feature-direction validation and a small gain.
  - **What happened:** New max (`~1.7852`), small but real increase.

- **Week 9**
  - **Idea:** Use RF-informed safe-region exploration with explicit bounds `x1,x2 in [0,0.15]`, `x3 in [0.06,0.20]`, `x4 in [0.20,0.30]`, `x5 in [0.34,0.46]`, `x6 in [0.55,0.70]`, and set `xi=0.03` to test whether small positive `x2` improves on the incumbent.
  - **Why I tried it:** RF indicated room to vary `x2` while preserving strongest structural prior on `x1`.
  - **What I expected:** Controlled non-incumbent gain without basin jump.
  - **What happened:** New max (`~1.8559`), clear improvement.

- **Week 10**
  - **Idea:** Continue safe-region exploration by fixing the strongest prior coordinates (`x1=0`, `x6≈0.61`) and making a meaningful local move only in `x2..x5` (including `x2` up to `0.15`) to avoid incumbent re-submission.
  - **Why I tried it:** Week 9 validated that modest non-incumbent local moves were effective.
  - **What I expected:** Another incremental gain if local slope remained positive.
  - **What happened:** New max (`~2.1028`), largest late-phase jump.

- **Week 11**
  - **Idea:** Relax earlier hard fixes on `x1/x6` slightly, keep the same local neighborhood around the Week 10 best, and use exploitative ranking to test whether small coordinate freedom can beat the incumbent.
  - **Why I tried it:** After strong gains, I tested whether mild constraint relaxation could reveal a nearby better combination.
  - **What I expected:** Potential final lift above Week 10.
  - **What happened:** Lower (`~1.9004`), so Week 10 remained best.

## Small but Important BO Mechanics I Used

- **ARD lengthscales for feature influence:**
  - Used from the start to identify which dimensions mattered most and which could be deprioritized.
  - Practical goal: guide constraint design and search bandwidth per feature.

- **Practical-range constraints (anti-corner control):**
  - Applied early to prevent unrealistic extreme hyperparameter settings.
  - Practical goal: keep recommendations implementable and reduce brittle corner solutions.

- **EI exploration tuning (`xi`) by phase:**
  - Lower `xi` in exploitation phases; higher `xi` when escaping local misses.
  - Practical goal: dynamically balance local refinement and basin escape.

- **Structured local bounds (ridge priors):**
  - Repeatedly constrained search around observed high-performing coordinates (`x4/x5/x6` bands, low `x1/x2/x3` zones).
  - Practical goal: preserve known good structure while probing nearby alternatives.

- **Diversity/repulsion on selected dimensions:**
  - Added on `(x4, x6)` in mid-phase to prevent candidate collapse.
  - Practical goal: sample multiple nearby modes rather than one repeated optimum.

- **Model comparison (GP vs RF) for directional checks:**
  - Used in later phase to validate whether GP-only guidance might be biased.
  - Practical goal: improve confidence in which coordinates to relax/fix.

- **Safe-region exploratory policy (late phase):**
  - Held strongest prior fixed (e.g., `x1=0`) while perturbing less-certain dimensions.
  - Practical goal: make informative non-incumbent moves with bounded risk.

## Short Final Takeaway

- Function 7 improved through **progressive constraint tuning**: early practical controls, mid-phase ridge-focused EI, then RF-informed safe exploration.
- The most effective pattern was **keep the proven structure, relax one thing at a time, and verify causally with the next evaluation**.
