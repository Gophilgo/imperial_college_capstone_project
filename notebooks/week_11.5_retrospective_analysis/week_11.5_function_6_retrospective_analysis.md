# Function 6 - 11-Week Strategy Overview

## Problem Summary (Plain English)

- **Task:** Choose `x1, x2, x3, x4, x5` to maximize recipe score `y` (scores are negative by design, so closer to zero is better).
- **What made this hard:** The function was noisy and high-dimensional, with interactions between ingredients and no simple linear pattern.
- **Practical meaning:** I needed realistic recipe constraints (avoid extreme boundaries), then robust local modeling when GP-only guidance became unstable.

## What EI-based BO Means (Simple)

- **BO idea:** Fit a GP surrogate from observed recipes, then pick next recipe via an acquisition function.
- **EI idea:** EI ("expected improvement") scores expected gain over the current best.
- **Why EI helps:** It balances:
  - **high predicted mean** (exploit),
  - **high uncertainty** (explore).
- **`xi` in simple terms:** Higher `xi` explores more; lower `xi` exploits more.

## What the Ensemble + Reweighted UCB Means (Simple)

- **Ensemble idea:** Combine GP and Random Forest predictions to reduce reliance on one model class.
- **Why:** GP gives smooth uncertainty-aware structure; RF is more robust when the surface is irregular/noisy.
- **UCB reweighting in this setup:**
  - `beta` controls weight on uncertainty (higher `beta` -> more exploration),
  - `gamma` controls penalty from model disagreement (higher `gamma` -> avoid points where GP/RF disagree strongly).
- **Practical interpretation:** Increasing `beta` while lowering `gamma` allowed controlled non-incumbent moves without over-penalizing disagreement.

## Week-by-Week

- **Week 1**
  - **Idea:** Start with GP + UCB in full 5D space.
  - **Why I tried it:** No clear linear structure from EDA, so uncertainty-aware BO was a sensible baseline.
  - **What I expected:** A useful first directional signal in complex space.
  - **What happened:** Strong initial point (`~ -0.6776`), became early best.

- **Week 2**
  - **Idea:** Move from boundary-prone UCB behavior to EI + boundary penalties.
  - **Why I tried it:** Week 1 showed unconstrained BO could suggest impractical extremes.
  - **What I expected:** More realistic candidate recipes and stable local gains.
  - **What happened:** Slight new max (`~ -0.6699`).

- **Week 3**
  - **Idea:** Keep EI with very low `xi` and exploit near the best region.
  - **Why I tried it:** Early improvements suggested local refinement was working.
  - **What I expected:** Incremental improvement by staying in-basin.
  - **What happened:** New max (`~ -0.6254`).

- **Week 4**
  - **Idea:** Continue EI + boundary penalty with minor exploitation tweaks.
  - **Why I tried it:** Consecutive gains justified method consistency.
  - **What I expected:** Another moderate local gain.
  - **What happened:** New max (`~ -0.6177`).

- **Week 5**
  - **Idea:** Allow a bit more movement in EI-selected point while staying practical.
  - **Why I tried it:** I wanted to avoid over-conservative micro-steps and test a broader local trade-off.
  - **What I expected:** Either clear gain or a signal that I had moved too far.
  - **What happened:** Large new max (`~ -0.4432`), biggest jump so far.

- **Week 6**
  - **Idea:** Keep EI + boundary penalty but intentionally raise exploration slightly (`xi` toward `0.005-0.01`), tighten boundary margin (`~0.12`), and test a nearby non-identical profile (`0.480, 0.110, 0.650, 0.900, 0.080`) instead of reusing the incumbent recipe.
  - **Why I tried it:** After a large gain, I tested whether nearby alternatives could outperform the incumbent.
  - **What I expected:** Either another gain or a boundary of the basin.
  - **What happened:** Very poor point (`~ -1.3625`), clear miss.

- **Week 7**
  - **Idea:** Explicitly re-center search around the Week 5 incumbent with a tight local box (`x1: 0.44-0.50`, `x2: 0.08-0.12`, `x3: 0.58-0.70`, `x4: 0.88-0.92`, `x5: 0.03-0.10`) and use EI + jitter/diversity so candidates stay in-basin but are not duplicates.
  - **Why I tried it:** Week 6 miss showed I needed to stay local and avoid large drifts.
  - **What I expected:** Recovery toward incumbent performance.
  - **What happened:** Still worse (`~ -0.6509`), no recovery yet.

- **Week 8**
  - **Idea:** Deliberately increase exploration: widen the local box (`x1: 0.38-0.52`, `x2: 0.06-0.14`, `x3: 0.50-0.78`, `x4: 0.84-0.96`, `x5: 0.02-0.15`), raise EI exploration (`xi ~ 0.025`), increase jitter (`~0.01`), and add ~400 random interior candidates (`[0.1, 0.9]^5`) to search for alternate basins.
  - **Why I tried it:** Two weak results suggested possible local overfitting or missed alternate pockets.
  - **What I expected:** Discover a better basin or at least improve model information.
  - **What happened:** Still weak (`~ -0.6441`), no gain.

- **Week 9**
  - **Idea:** Replace GP-only ranking with an ensemble candidate score from GP + RF: use both models' mean predictions plus an uncertainty/disagreement signal, then rank feasible local candidates by ensemble UCB instead of trusting one GP surface.
  - **Why I tried it:** GP-only recommendations had become unreliable across recent weeks.
  - **What I expected:** Better robustness under noise and irregular local structure.
  - **What happened:** New max (`~ -0.4351`), confirming the pivot.

- **Week 10**
  - **Idea:** Keep the ensemble policy, explicitly remove incumbent fallback, and reweight decision terms (`beta = 0.45`, `gamma = 0.02`) so uncertainty contributes more while disagreement penalty contributes less, forcing a genuine non-incumbent local move.
  - **Why I tried it:** I wanted to escape conservative behavior that kept re-selecting near-incumbent points.
  - **What I expected:** A meaningful non-incumbent local move with controlled risk.
  - **What happened:** New max (`~ -0.3265`), strong improvement.

- **Week 11**
  - **Idea:** Keep the same no-fallback + reweighted-UCB policy and make a controlled in-basin coordinate adjustment (the proposed move mainly changes `x2` from `0.150000` to `0.183333` while keeping other coordinates fixed) to test whether the local slope remains positive.
  - **Why I tried it:** Week 10 showed the policy could break plateau behavior.
  - **What I expected:** Another incremental local gain.
  - **What happened:** New max (`~ -0.1801`), best overall.

## Small but Important BO Mechanics I Used

- **Boundary penalties (realism constraints):**
  - Introduced early (Week 2 onward) to avoid impractical extreme ingredient values.
  - Practical goal: keep suggested recipes physically/plausibly usable.

- **Multiple random restarts in acquisition optimization:**
  - Used to reduce optimizer sensitivity to poor initializations in 5D.
  - Practical goal: improve chance of finding better EI/UCB optima.

- **Local-box + jitter/diversity controls:**
  - Used around incumbent (especially Weeks 7-8) to avoid exact duplicates.
  - Practical goal: gather new local information without drifting far.

- **`xi` tuning by phase (EI exploration knob):**
  - Lower `xi` in exploitation phases; higher `xi` in exploratory recovery phase.
  - Practical goal: match exploration intensity to current confidence.

- **Ensemble disagreement handling (`beta`, `gamma`):**
  - In late phase, higher `beta` increased useful uncertainty exploration; lower `gamma` reduced over-penalization of model disagreement.
  - Practical goal: allow informative non-incumbent moves while staying in-basin.

- **No-incumbent-fallback policy (late phase):**
  - Applied from Week 10 to prevent conservative reversion to incumbent-like points.
  - Practical goal: force BO to test a genuine next-best local candidate.

## Short Final Takeaway

- Function 6 improved most when I moved from **basic GP-EI** to a **robust ensemble local policy** with explicit uncertainty/disagreement controls.
- The key process lesson was **constraint and policy tuning over time**: practical boundaries first, then model robustness (`GP+RF`), then decision-rule changes (`beta/gamma`, no fallback) to escape plateaus.
