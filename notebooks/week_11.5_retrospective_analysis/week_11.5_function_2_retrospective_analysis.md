# Function 2 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2` to maximize a noisy log-likelihood-style score `y`.
- **What made this hard:** The function looked multi-modal (more than one promising region), and repeated evaluations suggested meaningful noise.
- **Practical meaning:** A single "best-looking" area was not enough; I had to balance local refinement with tests of alternative regions.

## What EI-based BO Means (Simple)

- **BO idea:** Fit a GP surrogate to observed points, then use an acquisition function to choose the next experiment.
- **EI idea:** EI ("expected improvement") asks: "How much improvement over current best do I expect here?"
- **Why EI helps:** It balances:
  - **high predicted mean** (exploit good areas), and
  - **high uncertainty** (explore places that could surprise upward).
- **`xi` in simple terms:** Higher `xi` pushes more exploration; lower `xi` stays more local/conservative.

## What Thompson Sampling Means (Simple)

- **Core idea:** Instead of picking the best point from one mean prediction, sample one plausible function from the GP posterior and optimize that sample.
- **Why useful in noise:** It naturally captures uncertainty and avoids over-trusting one noisy observation.
- **In plain words:** EI asks "best expected gain?"; Thompson asks "if this plausible world were true, where would I go?"

## Week-by-Week

- **Week 1**
  - **Idea:** Use manual, hypothesis-driven probing and test a high-`x1` region away from the obvious incumbent, explicitly checking whether a second promising branch existed outside the current top point.
  - **Why I tried it:** Early plots suggested high `x1` mattered strongly, with hints of more than one good zone.
  - **What I expected:** A strong signal about whether the upper-right/low-`x2` branch was truly competitive.
  - **What happened:** Good point (`~0.4588`), but not a new max.

- **Week 2**
  - **Idea:** Use GP + UCB and add diversity-aware candidate selection instead of taking only top clustered points, so the top proposals were forced to cover multiple parts of the space.
  - **Why I tried it:** I wanted BO candidates to cover several promising zones rather than collapse into one tight patch.
  - **What I expected:** Better global coverage and lower risk of missing a second mode.
  - **What happened:** Solid score (`~0.4688`), still not a new max.

- **Week 3**
  - **Idea:** Take a simple manual compromise point between two top regions (line-segment midpoint logic), i.e., test whether those two promising areas were connected by a useful bridge.
  - **Why I tried it:** The landscape looked multi-modal, so a middle test could probe connectivity between peaks.
  - **What I expected:** Either confirm a bridge region or falsify it quickly.
  - **What happened:** Strong score (`~0.5521`), but still below the best initial point.

- **Week 4**
  - **Idea:** Add a hard exploratory constraint to force search into an under-sampled top-left regime (`x1` low / `x2` high), as a deliberate falsification test of missed modes.
  - **Why I tried it:** I wanted a causal test of whether BO was over-committing to one mode too early.
  - **What I expected:** If a hidden mode existed there, this forced probe would reveal it.
  - **What happened:** Clear miss (`~-0.0485`), not a new max.

- **Week 5**
  - **Idea:** Recover from the forced-exploration miss by returning to local EI/UCB search near the known ridge, with mild novelty controls to avoid immediate point duplication.
  - **Why I tried it:** After Week 4, I expected value from re-centering near known positives while still keeping mild novelty controls.
  - **What I expected:** A safer rebound point near the incumbent basin.
  - **What happened:** Partial recovery (`~0.4712`), still not a new max.

- **Week 6**
  - **Idea:** Probe vertically along the ridge boundary (`x2` near 1.0) with light jitter/diversity to avoid duplicates, so I could test boundary upside without abandoning the `x1 ~ 0.7` corridor.
  - **Why I tried it:** I expected boundary uncertainty could hide gains without leaving the good `x1 ~ 0.7` corridor.
  - **What I expected:** Either a boundary improvement or clearer evidence to move back below the boundary.
  - **What happened:** Weaker point (`~0.3100`), not a new max.

- **Week 7**
  - **Idea:** Stay near the ridge but keep local exploration active and avoid stacking at nearly identical points, using distance/diversity rules to keep each evaluation informative.
  - **Why I tried it:** I wanted controlled exploration around the incumbent zone rather than another far jump.
  - **What I expected:** A better trade-off between local mean quality and uncertainty.
  - **What happened:** New max (`~0.6703`), important recovery.

- **Week 8**
  - **Idea:** Step slightly off the `x2=1` boundary (`~0.69, 0.90`) to test whether the true ridge sat just below the edge rather than exactly on the saturation boundary.
  - **Why I tried it:** I expected boundary hugging might be too rigid; a small inward move could improve robustness.
  - **What I expected:** Similar or better score with less boundary sensitivity.
  - **What happened:** Reasonable but lower (`~0.6255`), not a new max.

- **Week 9**
  - **Idea:** Switch from EI to Thompson Sampling because repeated points showed large output variability, i.e., use posterior-sample decisions instead of trusting one posterior mean surface.
  - **Why I tried it:** The same/near-same region produced noticeably different values, so I needed a method that handles uncertainty more directly.
  - **What I expected:** More robust decision-making under noise and less overconfidence in single mean estimates.
  - **What happened:** New max (`~0.7457`), best result in the 11-week run.

- **Week 10**
  - **Idea:** Keep Thompson Sampling and take a small local move near the incumbent, so I refined within the validated basin while preserving stochastic robustness to noise.
  - **Why I tried it:** I wanted method consistency and incremental refinement after the Week 9 jump.
  - **What I expected:** Stabilize around high-performing ridge points.
  - **What happened:** Not a new max (`~0.6416`), but stayed in the strong region.

- **Week 11**
  - **Idea:** Continue Thompson-led local search with controlled stochastic exploration near the incumbent ridge, prioritizing local robustness over a late global reset.
  - **Why I tried it:** I expected another local attempt was more valuable than a late global reset.
  - **What I expected:** Potential incremental gain if noise realization was favorable.
  - **What happened:** Not a new max (`~0.6362`), confirming a plateau below the Week 9 best.

## Small but Important BO Mechanics I Used

- **Distance to incumbent (local control):**
  - Used mainly in **Weeks 5-8** to keep candidates near the currently best ridge while still allowing movement.
  - Practical goal: avoid wasting evaluations in far-off low-value regions right after a miss.

- **Minimum distance to existing points (anti-duplicate rule):**
  - Applied in local-search phases (especially **Weeks 5-8**) so BO would not re-pick almost identical points.
  - Practical goal: each expensive evaluation should add new information, not just confirm the same location.

- **Diversity / jitter penalties:**
  - Added in **Weeks 2 and 5-8** to prevent top candidates from collapsing into one tiny cluster.
  - Practical goal: keep exploration within promising zones but still spread proposals enough to map local shape.

- **Hard exploratory constraints (region forcing):**
  - Used explicitly in **Week 4** to force a test in an under-sampled region.
  - Practical goal: causal test of "are we missing another mode?" (it failed, but answered the question quickly).

- **Kernel/noise modeling choices (noise awareness):**
  - Used Matérn + noise terms in GP throughout BO phases; noise handling became central by **Weeks 9-11**.
  - Practical goal: avoid overconfidence from noisy observations and choose points with uncertainty in mind.

- **Acquisition switch EI/UCB -> Thompson Sampling:**
  - Switched in **Week 9** after observing large variability at near-identical points.
  - Practical goal: sample-based decisions under uncertainty, instead of relying only on one posterior mean surface.

## Short Final Takeaway

- The most important pattern was **method adaptation to uncertainty**: early mixed EI/UCB exploration to map modes, then Thompson Sampling when noise became the dominant issue.
- The strongest gains came when I combined **local structure discipline** (stay near validated ridge) with **explicit anti-duplication / diversity controls** and switched acquisition strategy when assumptions changed.
