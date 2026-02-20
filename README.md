# KAPOW! Card Game

An original card game with a sophisticated AI opponent, built entirely in vanilla HTML/CSS/JavaScript. No frameworks, no dependencies, no build tools — just 4,600 lines of hand-crafted game logic.

**[Play the original](https://cpheterson.github.io/Kapow/)** | **[Play this fork](https://epheterson.github.io/Kapow/)**

---

## Hi Dad! 👋

I saw this and had to fork it immediately. Here's what I love about what you built, and what I'm adding on top.

### What's awesome in the original

- **It's a real game.** This isn't a tutorial project or a clone — KAPOW! is an original design with genuine strategic depth. The triad completion mechanic, power card modifiers, KAPOW wild risk/reward, and the going-out doubling penalty all create meaningful decisions every turn.
- **The AI is legitimately good.** Opponent threat modeling, synergy evaluation, discard safety analysis, future completion path counting — this is way beyond what you see in most browser games. It actually thinks about what *I* need and avoids feeding me useful cards.
- **"Understand AI's Move" is genius.** I've never seen this in a card game. Showing players WHY the AI made each decision turns the game into a learning tool. This alone makes it worth sharing.
- **The banter system gives the AI personality.** 100+ context-aware messages across 17 categories. "Thanks, this is exactly what I needed" when it grabs my discard, "Much appreciated!" when completing a triad. Makes it feel like playing against someone, not something.
- **It's complete.** 10 rounds, scorecard, round-by-round breakdown, game over screen, log export. This is a finished product.

### What I changed (so far)

**Bug fixes:**
- **XSS in player name** — names with HTML characters (like `<b>Eric</b>`) would break the round-end and game-over screens. Added `escapeHTML()` helper to sanitize before innerHTML injection.
- **AI deadlock on empty piles** — if both draw and discard piles ran out, the AI's turn would never end and the game would silently freeze. Now it gracefully ends the turn.
- **Scoring null guard** — `applyFirstOutPenalty` was called with `null` when nobody went out. It happened to work by accident (`roundScores[null]` is `undefined`), but now has an explicit guard.

**New stuff:**
- **PWA / iOS web app support** — manifest.json, service worker, Apple meta tags, app icons. You can "Add to Home Screen" on iPhone and it runs as a standalone app with its own icon.
- **README** — you're reading it!

### What I want to add next

Honestly, I need the tutorial stuff before I can really play well. I figured out triads and drawing but the power cards, KAPOW swaps, and when to go out are not obvious at all without reading the code. So that's high on my list.

The other big one is **sound** — the game is completely silent and every card game that feels good (Solitaire, Balatro, UNO) has satisfying audio. Card flips, placement thunks, a chime on triad completion, something dramatic for KAPOW moments.

See the full [Roadmap](#roadmap) below for everything planned.

### A note on the modular files

There are separate `js/*.js` files (deck.js, hand.js, triad.js, etc.) alongside the main `kapow.js` bundle. The HTML only loads `kapow.js` — the modular files aren't used. I noticed they've drifted from the bundle in a few spots (scoring tie rules, KAPOW freeze checks, first-turn phase handling). Might be worth either wiring them up with a build step or removing them to avoid confusion. Happy to help with that if you want!

---

## How to Play

KAPOW! is played over **10 rounds**. Lowest total score wins.

### The Cards (118 total)

| Type | Count | Description |
|------|-------|-------------|
| **Fixed** | 96 | Values 0–12. Lower is better. |
| **Power** | 16 | Modifiers (-1/+1 or -2/+2) that stack on existing cards to change their value |
| **KAPOW!** | 6 | Wild cards — assign any value 0–12 to complete a triad. **25-point penalty** if left unresolved |

### Setup

Each player gets 12 cards arranged in **4 triads** (columns of 3). All start face-down. You reveal 2 cards to start.

### On Your Turn

1. **Draw** from the deck or discard pile
2. **Place** the card in your hand (swapping out an existing card) or **discard** it (then you must reveal a face-down card)
3. Power cards can be **stacked** as modifiers beneath existing cards
4. KAPOW! cards can be **swapped** between positions after placement

### Triad Completion

When all 3 cards in a triad form a **set** (same value) or a **run** (ascending/descending sequence), the triad is **discarded for zero points**. This is how you win — eliminate entire columns from your score.

### Going Out

Declare you're going out to end the round. All other players get one final turn. **Warning:** if you don't have the lowest score, your score is **doubled**.

### Scoring

- Each remaining card scores its face value (modified by any Power cards)
- Completed triads score 0
- Unfrozen KAPOW! cards score 25
- First-out penalty: doubled if you didn't have the strictly lowest score

## Architecture

Single-file IIFE bundle (`js/kapow.js`, ~192KB unminified):

| Section | What It Does |
|---------|-------------|
| AI Banter | 100+ contextual trash-talk messages across 17 categories |
| Deck/Hand | Card creation, Fisher-Yates shuffle, deal, hand management |
| Triads | Completion detection: sets, runs, KAPOW value assignment |
| Scoring | Hand evaluation, first-out doubling penalty, winner determination |
| Game State | State machine: setup → firstTurn → playing → finalTurns → scoring → gameOver |
| AI Engine | Multi-layered heuristic evaluation with opponent modeling |
| UI | DOM rendering, modals, animations, scorecard |
| Action Log | Turn-by-turn history with text export |

### The AI

The AI opponent is the most sophisticated part of the codebase:

- **Placement scoring** evaluates every possible position using value delta, triad completion potential, synergy with revealed cards, and KAPOW timing
- **Opponent threat modeling** tracks your near-complete triads, reveal ratio, and estimated score
- **Discard safety analysis** avoids feeding you cards that complete your triads
- **Go-out timing** considers hand value, unrevealed count, and whether it would get doubled
- **Pedagogical explanations** — click "Understand AI's Move" after any AI turn to see a full breakdown of its reasoning

## Known Issues

### Remaining bugs

| Severity | Issue | Location |
|----------|-------|----------|
| Medium | **AI `reveals` array not bounds-checked** — if AI hand somehow has <2 unrevealed cards, `reveals[1]` crashes | `kapow.js` L4256 |
| Low | **Round-end screen re-shown on every `refreshUI()` during scoring phase** — unnecessary DOM rebuilds | `kapow.js` L3789 |

### Code quality

| Issue | Detail |
|-------|--------|
| **Divergent codebases** | The modular `js/*.js` files and `kapow.js` have different behavior. The modular files are never loaded. |
| **No build system** | 192KB served unminified. With esbuild/rollup, this would be ~30KB gzipped. |
| **Inline onclick handlers** | Cards use `onclick="window._onCardClick(...)"` instead of `addEventListener` |

### Accessibility

- No ARIA labels on card elements
- No keyboard navigation for card selection
- No color-blind accommodations for card types

## Roadmap

Prioritized by impact-to-effort ratio:

### Up next

- [ ] **Rules / help button** — a "?" that opens a modal with condensed rules. Right now new players have zero guidance on triads, powersets, KAPOW swaps, or going out. This is the #1 barrier to sharing the game.
- [ ] **Sound effects** — card draw, place, flip, triad completion, KAPOW moment, round end. The single biggest missing piece for game feel.
- [ ] **Comic book font** — swap Segoe UI for Bangers/Luckiest Guy on title, KAPOW cards, headings. Instant personality.

### Polish

- [ ] **Card movement animations** — fly cards from pile to hand to position instead of instant swap
- [ ] **Card flip animation** — CSS 3D rotateY for reveals
- [ ] **Confetti on triad completion** — canvas particle burst
- [ ] **Screen shake on KAPOW events** — CSS keyframes + class toggle
- [ ] **AI banter as speech bubbles** — move from sidebar to overlay near AI hand, typewriter effect
- [ ] **Table felt texture** — CSS noise pattern + vignette overlay
- [ ] **AI speed toggle** — `AI_DELAY` is hardcoded at 1500ms, add Normal/Fast/Instant
- [ ] **Score count-up animation** — animate numbers at round end

### Features

- [ ] **Interactive tutorial** — first-game guidance with tooltip arrows and AI narration
- [ ] **Difficulty levels** — beginner (random/greedy), normal (current), expert (deeper eval)
- [ ] **Persistent stats** — win/loss, streaks, best round via localStorage
- [ ] **Undo button** — snapshot state before each action, restore on undo
- [ ] **Probability hints** — show "outs" count on near-complete triads
- [ ] **Achievement badges** — "Shutout" (0-point round), "Comeback Kid", "KAPOW Master"
- [ ] **Share game summary** — score breakdown + play link for texting friends
- [ ] **Consolidate codebases** — unify modular files with kapow.js, add build step
- [ ] **Multiplayer** — game state already supports N players

## Development

No build tools needed. Just open `index.html` in a browser or serve locally:

```bash
python3 -m http.server 8000
# Then open http://localhost:8000
```

### Project Structure

```
Kapow/
├── index.html              # Entry point (loads kapow.js)
├── manifest.json           # PWA manifest
├── sw.js                   # Service worker (offline caching)
├── css/
│   └── styles.css          # All styles (~300 lines)
├── js/
│   ├── kapow.js            # Production bundle (4,606 lines)
│   ├── ai.js               # AI heuristics (modular, NOT loaded)
│   ├── deck.js             # Card/deck creation (modular)
│   ├── gameState.js        # State machine (modular)
│   ├── hand.js             # Hand management (modular)
│   ├── main.js             # Entry point (modular)
│   ├── rules.js            # Validation (modular)
│   ├── scoring.js          # Scoring (modular)
│   ├── triad.js            # Triad completion (modular)
│   └── ui.js               # UI rendering (modular)
├── icons/
│   ├── icon.svg            # Source icon
│   ├── icon-512.png        # PWA icon (512x512)
│   ├── icon-192.png        # PWA icon (192x192)
│   └── apple-touch-icon.png # iOS home screen icon
└── ai-banter-messages.txt  # Banter message reference
```

## Credits

Original game by [cpheterson](https://github.com/cpheterson)

Fork by [epheterson](https://github.com/epheterson) — adding polish, PWA support, sound, and tutorials
