# KAPOW! Card Game - Version Log

## Version History

### 02-23-2026 v5
**Fixed within-triad KAPOW swaps not detecting KAPOW in powersets.**
- Critical bug extension of v4 fix: KAPOW swaps only worked when KAPOW was solo
- If AI completed a triad with KAPOW in a powerset (KAPOW + Power modifier), swaps were skipped
- Example from R1 T6: AI had [KAPOW, P1(-1)], 8, 7 and completed, but didn't attempt burial swaps
- Fixed three functions to detect KAPOW in both solo and powerset forms:
  - `hasRevealedKapow()`: Now checks `posCards.length > 0` instead of `=== 1`
  - `swapKapowCard()`: Removed `sourceCards.length !== 1` check, now allows length 1 or 2
  - `aiStepWithinTriadSwap()`: Now checks `posCards.length > 0` instead of `=== 1`
- KAPOW in powersets [KAPOW, P1] or [KAPOW, P2] are now properly detected and swappable
- Swaps move entire position: KAPOW alone OR KAPOW+modifier as a unit
- Commit: 597dd67

### 02-23-2026 v4
**Fixed AI within-triad KAPOW swaps not triggering on triad completion.**
- Critical bug: AI would complete a triad with a revealed KAPOW and immediately discard it
- Example from R4 T4: AI placed a card completing a triad with KAPOW in top position
- Instead of swapping KAPOW to middle/bottom to bury it, AI discarded triad, leaving KAPOW on discard pile
- Root cause: Within-triad KAPOW swap phase only enabled for human player (currentPlayer === 0)
- Fixed three placement handlers to enable within-triad swaps for both human and AI:
  - `handlePlaceCard()`: check for KAPOW + completion regardless of player
  - `handleAddPowerset()`: same fix for Power card modifier placements
  - `handleCreatePowersetOnPower()`: same fix for powerset creation
- Now AI strategically buries KAPOW cards before triad discard, matching human player capability
- Commit: faf098c

### 02-23-2026 v3
**Reverted incorrect Power card scoring fix (added in v3 mistakenly then reverted in 889f035).**
- Power cards were briefly treated as scoring 0 points (incorrect)
- Reverted: Power cards correctly score their face value (1 for P1, 2 for P2)
- This was causing R1 T8 bug where P1 in completed triad incorrectly had 0 value
- Correct scoring: solo Power = face value; in powerset = face + modifier
- Commit: 889f035

### 02-23-2026 v2
**Implemented AI within-triad KAPOW swaps with oscillation prevention.**
- AI now performs strategic within-triad KAPOW swaps when completing triads
- Evaluates all three positions and selects optimal burial depth (bottom > middle > top)
- Adds bonus for moving KAPOW away from exposed top position
- Prevents oscillation by tracking swap history (never swaps KAPOW back to previous position)
- Includes detailed explanation in "Understand AI's Move" modal showing swap strategy
- AI recursively swaps KAPOW deeper until no better position is available
- After within-triad swaps complete, proceeds to discard triad and cross-triad KAPOW swaps
- Commit: e829ed4

### 02-23-2026 v1
**New session begins on 02-23-2026.**
- Version counter resets to v1 for new date
- All changes from 02-21-2026 carried forward

### 02-21-2026 v3
**Redesigned powerset value display to appear directly on card.**
- Powerset values now display on the fixed value card instead of below it
- Format: "Powerset = X" where X is the total value (card + all modifiers)
- Orange font (#ff9800), 14px, bold, centered above card bottom
- Prevents card layout shift caused by below-card powerset info
- Cleaner UI with improved readability for powerset values
- Removed individual modifier display and calculation details
- Commit: cde90c8

### 02-21-2026 v2
**Removed frozen KAPOW property (deprecated feature).**
- KAPOW cards are never frozen; this feature was removed from game long ago
- Cleaned up all references to isFrozen and assignedValue properties
- Simplified KAPOW checks from 'type === kapow && !isFrozen' to 'type === kapow'
- Removed triad freezing logic and KAPOW reset when discarded
- Code now reflects actual game mechanics where KAPOW is always wild (0-12)
- Commit: 832e09e

### 02-21-2026 v1
**Implemented within-triad KAPOW swaps for strategic card positioning.**
- When a placement completes a triad containing a revealed KAPOW, player can now swap it within that triad before discard
- Prevents KAPOW from being immediately exposed on the discard pile's top position
- New swap phase enters after placement but before triad discard
- Button text changes to "Discard Triad and End Turn" during within-triad swap phase
- Message box highlights with swap-phase styling
- New helper functions: `hasRevealedKapow()`, `completeWithinTriadSwap()`
- Modified `findSwapTargets()` with optional scope restriction parameter
- AI unchanged: uses normal cross-triad swap logic after discard
- Commit: 1011155

### 02-20-2026 v3
**Fixed AI explanation to avoid nonsensical future completion paths when going out.**
- When placement triggers going out, explanation now focuses on going-out decision
- Removed mention of "completion paths" that will never be needed post-game
- Explanation now clarifies why the timing was right to end the round

### 02-20-2026 v2
**Strengthened discard safety penalty for opponent-completion cards.**
- Increased scaling factor for replaced-card discard penalty from 0.4 to 1.0
- Penalty now directly proportional to danger level (stronger deterrent)
- Prevents AI from discarding cards that would help opponent complete triads
- Example: replacing 9 with 3 now scores -2 instead of +3 when 9 completes opponent's triad

### 02-20-2026 v1
**Initial revision with software revision footer added.**
- Added revision footer to lower right corner of page
- Format: `02-20-2026 v1` (date v{version})
- Created VERSION_LOG.md for version tracking

---

## Version Numbering Convention

- Format: `MM-DD-YYYY vN` where N is a sequential number (1, 2, 3, etc.)
- Each new date resets the version counter to v1
- Multiple versions on the same day are incremented (v1, v2, v3, etc.)
- The revision text in the HTML footer (`index.html` line ~150) should be updated to match the latest version

## How to Increment

When making changes that warrant a version bump:

1. Determine today's date in MM-DD-YYYY format
2. Check if today's version already exists (e.g., is there a v1, v2, etc.?)
3. Increment accordingly:
   - If new date: use `MM-DD-YYYY v1`
   - If same date: increment to next number (e.g., `02-20-2026 v2`)
4. Update `id="revision-text"` in `index.html` with the new version string
5. Add an entry to this VERSION_LOG.md file describing the changes
6. Commit the changes with the version in the commit message

---

### 02-23-2026 v6
**Fixed powerset value display and improved AI powerset placement strategy.**
- **Powerset display fix:** Power cards on top of other Power cards now show "Powerset = x" in red bold font (e.g., P1 on top of P2 now displays the effective value)
- **AI powerset placement strategy improvements:**
  - Added +8 bonus per KAPOW card in other positions of the same triad (recognizes opportunity for defensive swaps)
  - Added +6 positional bonus for placing powersets in top position when KAPOW burial opportunities exist
  - Example: AI placing P1 on P2 in top position now scores higher when triad also has a KAPOW, encouraging strategic positioning for future swaps
  - These changes encourage the AI to facilitate KAPOW burial through within-triad swaps after triad completion
- Fixed animation issue where within-triad KAPOW swaps followed by discard now properly animates cards disappearing one at a time
- Commit: (pending)

### 02-24-2026 v1
**AI within-triad KAPOW swap now verified to keep triad complete; human swap validated too.**
- AI `aiStepWithinTriadSwap`: replaced positional-score-only logic with simulation-based validation
  - Only attempts swaps when KAPOW is at top position (middle/bottom already buried — no action needed)
  - Simulates each candidate swap via `isTriadComplete()` before committing
  - Sets (K,x,x): swap with either position always valid — correctly confirmed
  - Runs (K,m,b): swap with bottom valid except when mid=0 (ascending) or mid=12 (descending) — correctly caught
  - Swapping K with middle in a run typically breaks it — now properly skipped
  - Still prefers deepest burial (bottom=15 > middle=10)
- Human `_onCardClick` within-triad swap: added same simulation check
  - If a swap would break the triad, shows message "That swap would break the triad! Choose a different position."
  - Prevents player from accidentally clicking an invalid target position
- Commit: (pending)

### 02-24-2026 v2
**Fixed within-triad KAPOW swap: one swap, no loop; KAPOW top-position penalty exempted on triad completion.**
- `aiStepWithinTriadSwap` rewritten to perform exactly one swap then immediately discard:
  - No longer recurses via `setTimeout` — eliminates any risk of infinite swap loops
  - Prefers bottom burial first, falls back to middle; takes the first valid option and stops
  - If KAPOW is already at middle or bottom (already buried), skips straight to discard
  - Swap validity still confirmed via `isTriadComplete()` simulation
  - `withinTriadSwapHistory` removed — no longer needed since only one swap is ever made
- `aiScorePlacement` defensive top-position penalty now exempted when placement completes the triad:
  - Previously a KAPOW drawn to the top slot of a completing triad incurred a -17 penalty
  - This suppressed valid triad-completing KAPOW placements at the top position
  - The penalty is unnecessary when the triad completes — the within-triad swap buries KAPOW before discard
  - New `placementCompletesTriad` flag captured during simulation and checked in the penalty block

### 02-24-2026 v3
**Fixed human within-triad KAPOW swap hanging after swap.**
- After a successful swap, the game showed "Swap again, or Discard Triad and End Turn" indefinitely
- Root cause: `hasRevealedKapow` is always true (KAPOW is still in the triad, just repositioned)
  so the code always offered another swap instead of auto-proceeding to discard
- Fix: after the swap, check the KAPOW's new position:
  - If buried (middle or bottom) → auto-proceed to `completeWithinTriadSwap` immediately
  - If still at top (swap to buried position wasn't valid) → show "Swap again" message for one more attempt
- Mirrors the AI's one-swap-and-done logic from v2

### 02-24-2026 v4
**Fixed AI not taking its turn after human within-triad KAPOW swap + discard animation.**
- Root cause: in `completeWithinTriadSwap`, `refreshUI()` was called before `endTurn(state)`
  inside the animation callback. At that point `currentPlayer` was still the human, so the
  AI trigger check (`!isHumanTurn`) was false and `playAITurn` was never scheduled.
  After `endTurn` switched `currentPlayer` to the AI, no further `refreshUI()` was called.
- Fix: swap the order to `endTurn(state)` first, then `refreshUI()` — so when `refreshUI`
  runs, `currentPlayer` is already the AI and the `setTimeout(playAITurn, 1000)` fires.

### 02-24-2026 v5
**Fixed `aiCountFutureCompletions` to be KAPOW-aware for 3-revealed triads.**
- Root cause: when scoring placement of a card into a face-down slot completing [x, K, y],
  `aiCountFutureCompletions([x, 25, y])` treated KAPOW as fixed value 25, finding zero
  paths since no set/run includes 25. This caused the 3-revealed bonus to fire as -20
  (zero paths penalty) instead of rewarding the many real completion paths.
- Example: [5, K, 11] previously scored 0 future paths → -20 penalty. In reality, e.g.
  replacing pos 0 (5) with 9 and K=10 gives [9,10,11] ascending run — a valid path.
- Fix: `aiCountFutureCompletions` now detects when one value is 25 (KAPOW placeholder)
  and for each candidate replacement value, tests all 13 KAPOW assignments (0-12) to find
  any combination that completes the triad. Mirrors `aiAnalyzeTriad`'s existing KAPOW handling.
- Impact: placing a card into a face-down slot to create [x, K, y] now correctly scores
  the real completion path count, making it competitive with replacing existing revealed cards.

### 02-24-2026 v6
**Fixed hang after AI within-triad KAPOW swap: missing refreshUI in else branch of completeWithinTriadSwap.**
- Root cause: after AI's placement step discards the triad (via `checkAndDiscardTriads`), it then
  calls `aiStepWithinTriadSwap` → `completeWithinTriadSwap`. At this point the triad is already
  marked discarded, so `newlyDiscarded.length === 0` and the `else` branch fires: `endTurn(state)`
  with no `refreshUI()` following it. The next player's turn never triggered.
- Fix: added `refreshUI()` after `endTurn(state)` in the `else` branch, matching the animation
  callback path which already had this correct ordering.
- Same root cause as v4 (human path) — the else branch was missed in that fix.

### 02-24-2026 v7
**Fixed AI hang after within-triad KAPOW swap: aiTurnInProgress never cleared.**
- Root cause: `aiStepWithinTriadSwap` → `completeWithinTriadSwap` → `endTurn` bypasses
  `aiStepCheckSwap`, which is the only place `aiTurnInProgress = false` was set.
  With the guard stuck at `true`, the next AI turn could never start.
- Fix: `completeWithinTriadSwap` now sets `aiTurnInProgress = false` before `endTurn`
  in both the animation callback path and the else (no-animation) path.
- This is the definitive fix for the within-triad swap hang — previous fixes (v4, v6)
  addressed `refreshUI` ordering but missed the `aiTurnInProgress` guard.

### 02-24-2026 v8
**Strengthened AI discard safety scoring to prevent gifting opponent triad-completing cards.**
- Root cause: discard score formula `safety * 0.05 - 2` produced a tiny range (-2 to +3).
  A card that directly completes an opponent triad (safety=25) only scored -0.75 — barely
  negative. Late in the round when placement scores also collapse, even marginal placements
  beat -0.75, but mediocre placements (e.g., score=-1) could lose to it, letting the AI
  discard a card the opponent immediately uses to win the triad.
- New formula: `(safety - 50) * 0.2 - 2` with extra penalty for safety < 30.
  - Safe discard (safety=100): +8 — chosen freely when no good placement exists
  - Neutral (safety=50): -2 — slight preference for placement over discard
  - Dangerous (safety=25): -9.5 — strongly avoids giving opponent completion cards
  - KAPOW (safety=15): -12 with extra -7.5 = -19.5 — almost never discarded
- Discard reason string now includes safety value for easier log debugging.

### 02-24-2026 v9
**Fixed Power card modifier (+/-) values overlapping "Powerset = x" label.**
- `.card-power-modifiers` repositioned to `position: absolute; bottom: 6px` so it anchors at the card bottom like fixed-card values — no longer rendered in the flex flow where it collided with the powerset label.
- `.powerset-value-on-card` raised slightly from `bottom: 26px` to `bottom: 30px` to ensure clean clearance above the modifier row.
- Result: Power cards now show "Powerset = x" in red above the +/- modifiers with no overlap.

### 02-24-2026 v10
**AI recognizes KAPOW-swap triad completions as offensive strategy.**
- Added one-step lookahead in `aiScorePlacement`: after simulating a card placement, if the triad is all-revealed but not yet complete, try all possible within-triad KAPOW swaps. If any single swap would complete the triad, award `+80 + existingPoints` bonus (nearly as valuable as direct completion at `+100 + existingPoints`).
- Same check added to the final-turn short-circuit path.
- Updated top-position KAPOW penalty exemption to also bypass when placement completes via KAPOW swap.
- Updated action reason threshold from `ps >= 100` to `ps >= 80` so KAPOW-swap completions log as "completes Triad X".
- Example: AI had [4,10,K!] and drew 8. Mid replacement saved 2 pts (score +29). Top placement makes [8,10,K!] → swap K! to mid → ascending run [8,9,10]. Now scores ~123 and is correctly chosen.

### 02-24-2026 v11
**Steepened discard safety penalty to prevent gifting triad-completing cards.**
- Root cause from R6 T26: AI drew 8, discarded it (safety=39), score -4.2. Best placement was -4.36. A margin of only 0.16 caused the mistake — the 8 completed Chuck's [9,8,8] triad.
- New two-segment formula: above safety=50, mild positive slope `(s-50)*0.15 - 2`. Below 50, steep negative `-(50-s)*0.4 - 2`, plus extra steepness below 40: `-(40-s)*0.4`.
- safety=39 → -6.8 (was -4.2), safety=25 → -18 (was -9.5), safety=80 → +2.5 (safe discards still work).

### 02-24-2026 v12
**Fix: KAPOW-swap completion bonus unfairly inflated vs direct completion.**
- Bug: `kapowSwapExistingPoints` summed ALL 3 triad positions including the placed slot. For a placed KAPOW (value=25), this added 25 to the bonus, making the swap path score higher than a direct completion at a different slot that correctly excluded the placed position.
- Example from R2 T13: AI had T2=[11,11,10] and drew KAPOW. T2-bottom gives direct completion [11,11,K!] → score 100+22=122. T2-middle gives KAPOW-swap completion [11,K!,10]→swap K! to top→[K!,11,10] = run → score 80+46=126 (46 included the placed KAPOW=25). Wrong: mid won over bottom.
- Fix: skip `posIdx` (the placed slot) when computing `kapowSwapExistingPoints`, matching exactly how direct completion's `existingPoints` is calculated.
- Corrected scores: T2-bottom direct=122, T2-middle swap=80+21=101. Bottom wins by ~21 — AI now correctly places KAPOW at bottom to complete T2 directly, discarding 32 pts instead of 28.

## Latest Version: 02-24-2026 v12
