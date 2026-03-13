# Function 4 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2, x3, x4` to maximize model quality score `y` in a noisy, multi-optima setting.
- **What made this hard:** The function had several local optima and occasional very sharp drops, so nearby points could still differ a lot.
- **Practical meaning:** I had to combine local exploitation near good basins with explicit diversity/noise-aware controls to avoid getting trapped.

## What EI-based BO Means (Simple)

- **BO idea:** Fit a GP surrogate from observed points, then choose next experiment with an acquisition score.
- **EI idea:** EI ("expected improvement") measures expected gain over current best.
- **Why EI helps:** It balances:
  - **high predicted mean** (exploit),
  - **high uncertainty** (explore).
- **`xi` in simple terms:** Higher `xi` explores more; lower `xi` stays more local/conservative.

## What Thompson Sampling Means (Simple)

- **Core idea:** Sample plausible functions from the GP posterior and pick points that win often across those samples.
- **Why useful here:** In noisy multi-modal settings, this is often more robust than trusting one smooth posterior mean surface.
- **In plain words:** EI asks "best expected gain now?"; Thompson asks "where would I go if this plausible version of reality were true?"

## Week-by-Week

- **Week 1**
  - **Idea:** Start with GP + UCB and add a low-mean-`x` penalty (`x_avg`) from EDA intuition, so acquisition favored uncertain/high-mean points that also stayed in the lower-average input region.
  - **Why I tried it:** Early data suggested lower average input values tended to correlate with better outcomes.
  - **What I expected:** A safe first BO move toward the low-average basin.
  - **What happened:** Very poor result (`~ -11.55`), showing the heuristic was too simplistic.

- **Week 2**
  - **Idea:** Keep BO/UCB but remove overconfidence in the "just keep values low" heuristic, letting all four coordinates re-balance instead of forcing one simplistic prior.
  - **Why I tried it:** Week 1 showed that pushing one intuition too hard could fail badly in 4D.
  - **What I expected:** Better-balanced candidate selection across all four coordinates.
  - **What happened:** Large recovery (`~ -0.058`), but still not near top regime.

- **Week 3**
  - **Idea:** Continue BO around the improving region found in Week 2, using nearby exploitation to test whether that basin had consistent positive slope.
  - **Why I tried it:** I expected incremental exploitation in that basin to keep improving.
  - **What I expected:** Another positive step if the basin was real.
  - **What happened:** Further improvement (`~ -0.014`), confirming the recovery direction.

- **Week 4**
  - **Idea:** Follow-up with another local move near the improving cluster, effectively stress-testing whether the apparent continuity held under another close perturbation.
  - **Why I tried it:** I expected nearby points might cross into positive territory.
  - **What I expected:** Small gain from local continuity.
  - **What happened:** Reversal (`~ -0.100`), indicating sharper local structure than expected.

- **Week 5**
  - **Idea:** Tighten local BO around the strongest cluster and exploit aggressively, i.e., narrow search to the incumbent neighborhood where the ridge signal looked strongest.
  - **Why I tried it:** Mixed Week 2-4 behavior suggested one narrow productive pocket might dominate.
  - **What I expected:** Potential breakout if local ridge was correctly centered.
  - **What happened:** Breakthrough new max (`~ 0.5518`) at `(0.4303, 0.3593, 0.3518, 0.3837)`.

- **Week 6**
  - **Idea:** Keep local exploitation near the new incumbent with controlled novelty, so proposals stayed in-basin but still moved enough to reveal local curvature.
  - **Why I tried it:** After a large jump, I wanted to map immediate neighborhood rather than jump globally.
  - **What I expected:** Near-best points and better local geometry understanding.
  - **What happened:** Strong but lower (`~ 0.3711`), confirming a good basin but not a new max.

- **Week 7**
  - **Idea:** Continue local EI with narrow-box constraints and anti-duplicate guardrails (minimum distance and jitter), to avoid collapsing onto the exact incumbent coordinates.
  - **Why I tried it:** I wanted to test if very local movement could recover the 0.55 peak reliably.
  - **What I expected:** Either reclaim near-peak values or prove the peak was very narrow.
  - **What happened:** Drop (`~ -0.0502`), signaling high local brittleness.

- **Week 8**
  - **Idea:** Keep local EI but add rougher/noisier GP assumptions (Matérn/WhiteKernel tuning) plus diversity/jitter penalties, so ranking became less over-smooth and less duplicate-prone.
  - **Why I tried it:** Week 7 drop suggested local overfitting or basin switching under noise.
  - **What I expected:** More robust local ranking and reduced collapse onto exact incumbent.
  - **What happened:** Partial recovery (`~ 0.2620`), still far below best.

- **Week 9**
  - **Idea:** Switch to Thompson Sampling for noisy multi-modal search, drawing posterior samples to choose candidates that were robust across plausible function realizations.
  - **Why I tried it:** I wanted a method less dependent on one acquisition surface in a function described as noisy with many local optima.
  - **What I expected:** More robust candidate choice under uncertainty and basin ambiguity.
  - **What happened:** Very poor evaluation (`~ -1.3668`), later interpreted as potentially affected by execution/log mismatch.

- **Week 10**
  - **Idea:** Keep constrained Thompson/GP process but re-anchor around incumbent structure, treating the prior bad run as unreliable and prioritizing in-basin recovery.
  - **Why I tried it:** I treated Week 9 as unreliable evidence and avoided overreacting with a full method reset.
  - **What I expected:** Return to positive basin without over-exploring.
  - **What happened:** Recovery to solid positive (`~ 0.3216`), third-best range.

- **Week 11**
  - **Idea:** Continue constrained Thompson-guided local search near the incumbent with the same local-box discipline, aiming for incremental improvement rather than another global swing.
  - **Why I tried it:** Remaining budget favored disciplined local attempts over broad exploration.
  - **What I expected:** Possible incremental gain if local sampling hit the right pocket.
  - **What happened:** Positive but lower (`~ 0.1888`), no new max.

## Small but Important BO Mechanics I Used

- **Local-box constraints around incumbent:**
  - Used heavily from **Weeks 5-8** to keep search near the best-known basin in 4D.
  - Practical goal: reduce expensive off-basin jumps after the Week 5 breakthrough.

- **Minimum distance / anti-duplicate filters:**
  - Applied in local EI phases to avoid re-proposing already tested points.
  - Practical goal: each evaluation should add new information, not repeat incumbent coordinates.

- **Band/drift penalties on selected coordinates:**
  - Used in late local EI phases (e.g., x2/x3 drift control, x3/x4 band preferences).
  - Practical goal: stay near empirically good ridge structure while still allowing movement.

- **Noise-aware GP kernel choices:**
  - Added/adjusted `WhiteKernel` and rougher Matérn settings in later local phases.
  - Practical goal: avoid over-smooth surrogate behavior in noisy local neighborhoods.

- **Diversity/jitter penalties:**
  - Used to prevent candidates from snapping back exactly to the incumbent.
  - Practical goal: maintain local exploration without abandoning the good basin.

- **Acquisition switch to Thompson Sampling:**
  - Introduced in **Week 9** for noisy multi-modal robustness, then kept with constraints.
  - Practical goal: handle uncertainty via posterior sampling rather than one deterministic acquisition ranking.

## Short Final Takeaway

- Function 4 required **adaptive local control**: one large win came from tight local exploitation, but sustaining that level was difficult due to noise and local-optima switching.
- The strongest process pattern was **explicit constraint engineering** (local boxes, anti-duplicates, drift/band controls) plus a method switch when uncertainty handling became central.
