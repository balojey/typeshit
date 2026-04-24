# 🧠 Project Architecture: Falling Word Typing Game

## 1. 🎯 Product Overview

A real-time typing game where words fall from the top of the screen. Players must type them before they reach the bottom. The system dynamically adapts difficulty based on performance and introduces layered gameplay — combos, multiple modes, multiplayer, and a learning engine.

The core thesis: **a typing training engine disguised as a game**. That's the moat.

---

## 2. 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Game Client (Browser)              │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌────────────────┐    │
│  │ Renderer │  │  Input   │  │  Game Loop     │    │
│  │ (Canvas) │  │ Handler  │  │  (rAF ~60fps)  │    │
│  └──────────┘  └──────────┘  └────────────────┘    │
│  ┌──────────┐  ┌──────────┐  ┌────────────────┐    │
│  │ UI Layer │  │ Diff.    │  │  Net Layer     │    │
│  │ (HUD)    │  │ Engine   │  │  (WS Client)   │    │
│  └──────────┘  └──────────┘  └────────────────┘    │
└─────────────────────────────────────────────────────┘
                        │ WebSocket / REST
┌─────────────────────────────────────────────────────┐
│                     Backend                         │
│  Auth │ Leaderboard │ Matchmaking │ Analytics       │
│  Word Service │ Session State (Redis)               │
└─────────────────────────────────────────────────────┘
                        │
┌─────────────────────────────────────────────────────┐
│                   Data Layer                        │
│  PostgreSQL (users, scores)  │  Redis (sessions)    │
│  Word Banks (JSON / DB)                             │
└─────────────────────────────────────────────────────┘
```

> ⚠️ CORRECTION: The original listed "physics" in the game loop. Falling words don't need a physics engine — they move at a constant or linearly accelerating speed along the Y axis. No collision detection, no gravity simulation. Calling it "physics" is misleading and could lead you to over-engineer with a library like Matter.js when you don't need it. Just `y += speed * deltaTime`.

---

## 3. ⚙️ Core Game Engine Design

### Game Loop

Use `requestAnimationFrame` (rAF), not `setInterval`. rAF syncs to the display refresh rate, pauses in most browsers when the tab is hidden, and avoids drift. The timestamp it passes is a `DOMHighResTimeStamp` measured from the page's time origin (not Unix epoch) — don't compare it to `Date.now()`.

```js
let lastTime = null;

function gameLoop(timestamp) {
  if (lastTime === null) {
    lastTime = timestamp; // skip first frame — deltaTime would be huge
    requestAnimationFrame(gameLoop);
    return;
  }

  const deltaTime = Math.min((timestamp - lastTime) / 1000, 0.1); // seconds, capped at 100ms
  lastTime = timestamp;

  update(deltaTime);
  render();

  requestAnimationFrame(gameLoop);
}
```

**Correct loop order:**
```
1. Calculate deltaTime
2. Update word positions (y += speed * deltaTime)
3. Check loss condition (word reached bottom)
4. Check input match against active words
5. Remove matched / expired words
6. Spawn new words (based on difficulty state)
7. Render frame
```

> ⚠️ CORRECTION: The original had "Check loss condition" at the end, after spawning. That's wrong — you should check if words have escaped before spawning new ones, so your difficulty engine has accurate state when deciding whether to spawn.

> ⚠️ BUG FIX (cross-check): Initializing `lastTime = 0` causes a massive deltaTime spike on the very first frame (timestamp is milliseconds since page load, not 0). Always initialize to `null` and skip the first frame. Also cap deltaTime at ~100ms to prevent words teleporting after the tab regains focus (rAF resumes and delivers a huge elapsed time).

---

### Word Object Model

```ts
interface Word {
  id: string;           // uuid or incremental
  text: string;
  x: number;            // canvas px
  y: number;            // canvas px
  speed: number;        // px per second (use deltaTime, not per-frame)
  lane: number;         // column index — prevents overlap
  difficulty: 1 | 2 | 3 | 4 | 5;
  category: string;     // 'tech' | 'finance' | 'general' etc.
  spawnedAt: number;    // timestamp — for reaction time tracking
  isMatched: boolean;
}
```

> ⚠️ CORRECTION: The original used `isActive` as the only state flag. You need at minimum `isMatched` (typed correctly) vs expired (fell off screen) — these are different outcomes and you need to distinguish them for scoring, analytics, and the learning layer. Also: speed should be in **px/second** not px/frame, otherwise your game runs differently on 144hz vs 60hz monitors.

> ➕ ADDITION: `lane` is critical. Without it, words will overlap on the X axis and become unreadable. Divide the canvas into N lanes and assign each word a lane on spawn.

---

### Input Matching Logic

The original suggested a trie. That's technically valid but almost certainly overkill for this use case. Here's the real tradeoff:

| Approach | When to use |
|---|---|
| Linear scan | < 20 active words on screen. Simple, fast enough. |
| Prefix trie | 50+ words, or if you want to highlight partial matches in real-time |
| Inverted index | If words can be multi-word phrases |

For MVP: linear scan with prefix highlighting is the right call.

```ts
// On each keystroke, update buffer and find prefix matches
function onKeyPress(key: string) {
  buffer += key;

  const match = activeWords.find(w => w.text === buffer);
  if (match) {
    removeWord(match);
    buffer = '';
    return;
  }

  // Highlight partial matches for UX feedback
  const partials = activeWords.filter(w => w.text.startsWith(buffer));
  if (partials.length === 0) buffer = ''; // no match possible, reset
}
```

> ➕ ADDITION: Reset the buffer when no word starts with the current input. This is the most important UX decision in the whole game — if you don't do this, players get stuck with a broken buffer and feel cheated. Some games require pressing Space or Escape to reset; that's worse UX.

> ➕ ADDITION: "Word locking" — once a player starts typing a word (partial match exists for only one word), lock the input to that word. Prevents ambiguity when two words share a prefix.

---

## 4. 🧩 The 5 Differentiators

---

### 1. 🧠 Adaptive Difficulty Engine

The original logic is directionally correct but too simplistic. Real adaptive difficulty needs to avoid oscillation (rapidly toggling hard/easy) and needs smoothing.

#### Inputs (track a rolling window, not instantaneous):
- WPM (words per minute) — rolling 10-word average
- Accuracy — rolling 20-keystroke window
- Reaction time — time from word spawn to first correct keystroke
- Miss rate — words that escaped vs total spawned

#### Outputs:
- `spawnInterval` (ms between new words)
- `wordLength` (character count range)
- `fallSpeed` (px/sec)
- `wordCount` (max simultaneous words on screen)

#### Corrected logic with smoothing:

```ts
// normalize: clamp value between min/max and return 0–1
function normalize(value: number, min: number, max: number): number {
  return Math.max(0, Math.min(1, (value - min) / (max - min)));
}

// Don't jump difficulty — use a score that moves gradually
function updateDifficultyScore(metrics: PlayerMetrics): number {
  const wpmScore    = normalize(metrics.wpm, 20, 120);          // 0–1
  const accScore    = normalize(metrics.accuracy, 0.5, 1.0);    // 0–1
  const reactScore  = normalize(metrics.reactionMs, 3000, 300); // 0–1 (lower ms = higher score)

  const raw = (wpmScore * 0.4) + (accScore * 0.4) + (reactScore * 0.2);

  // Exponential moving average — moves 10% toward new value per update
  return currentDifficulty + (raw - currentDifficulty) * 0.1;
}
```

> ⚠️ CORRECTION: The original's `if accuracy drops → reduce spawn rate` is too reactive. If you reduce difficulty the moment accuracy drops, players never feel challenged. Use a smoothed score and only adjust when the trend is sustained over N words.

> ➕ ADDITION: Separate difficulty axes. A player might be fast but inaccurate — they need shorter words at higher speed, not longer words. Don't collapse everything into one "difficulty level" number.

---

### 2. ⚡ Pressure System

The original is solid. A few additions:

#### Combo system — be precise about the math:
```
score = base_word_score * combo_multiplier
combo_multiplier = min(1 + (combo_count * 0.1), 4.0)  // cap at 4x
combo resets on: miss OR word escaping screen
```

#### Danger zone — make it mechanical, not just visual:
- When `active_word_count > threshold`, trigger "danger mode"
- In danger mode: spawn rate pauses (give player a chance to catch up)
- This is more fun than just adding visual effects — it creates a recoverable moment

> ➕ ADDITION: "Last word" tension — when only one word is left and it's near the bottom, slow it down by 20% and add a heartbeat sound. This is a well-known game feel trick (used in Tetris, Superhot, etc.) — perceived fairness increases dramatically.

---

### 3. 🎮 Game Modes System

The pluggable module idea is right. Here's a more concrete interface:

```ts
interface GameMode {
  id: string;
  name: string;
  spawnStrategy: (state: GameState) => Word | null;
  winCondition: (state: GameState) => boolean;
  lossCondition: (state: GameState) => boolean;
  scoreMultiplier: number;
  modifiers: ModeModifier[]; // e.g. 'no_backspace', 'timed', 'lives'
}
```

#### Modes (expanded):
- **Survival** — words fall faster over time, no win condition
- **Time Attack** — type as many words as possible in 60s
- **Boss Mode** — one long phrase, must complete it before it reaches bottom; boss has a health bar
- **Zen Mode** — no loss condition, adaptive speed, pure flow state
- **Hardcore** — no buffer reset hints, no partial highlighting, one miss = game over
- **Speedrun** — fixed word list, race to clear it fastest (async leaderboard friendly)

> ➕ ADDITION: Speedrun mode is extremely leaderboard-friendly because everyone types the same words — scores are directly comparable. Great for competitive/social features.

---

### 4. 📚 Learning Layer

The original is conceptually right. Here's how to actually implement it:

#### Word performance tracking:
```ts
interface WordStat {
  word: string;
  attempts: number;
  misses: number;       // typed wrong
  escapes: number;      // fell off screen
  avgReactionMs: number;
  lastSeen: number;     // timestamp
}
```

#### Spaced Repetition (SM-2 simplified):
Don't build your own — the algorithm is well-documented. Key idea: words you struggle with come back sooner. Words you nail get pushed further back.

```
// SM-2 core (verified against original Wozniak spec)
// ease_factor starts at 2.5, minimum 1.3
// q = quality of response (0–5 scale, map: typed_fast=5, typed_slow=3, missed=1)

if q >= 3:
  if repetitions == 0: interval = 1 day
  elif repetitions == 1: interval = 6 days
  else: interval = round(last_interval * ease_factor)
  repetitions += 1
else:
  repetitions = 0
  interval = 1

ease_factor = max(1.3, ease_factor + 0.1 - (5 - q) * (0.08 + (5 - q) * 0.02))
```

> ⚠️ CROSS-CHECK FIX: The previous version wrote `next_interval = last_interval * ease_factor` as the full formula. That's incomplete — SM-2 has three phases (first review, second review, then the multiplier kicks in) and the ease factor update formula is non-trivial. The ease factor floor is 1.3, not 0. Using the wrong formula means words never actually space out correctly.

> ⚠️ CORRECTION: The original listed spaced repetition as "optional." It's not optional if the learning layer is a core differentiator — it IS the learning layer. Without it, you're just tracking stats with no feedback loop.

> ➕ ADDITION: Category tagging on words enables "focus sessions" — e.g., "practice finance vocabulary for 5 minutes." This is a direct monetization angle (B2B: companies pay for employees to learn domain vocabulary).

---

### 5. 🌐 Multiplayer System

The deterministic seed approach in the original is correct and elegant. Here's the full picture:

#### Deterministic spawning:
```ts
import seedrandom from 'seedrandom';

// Both clients use the same seed → same word sequence, same timing
// seedrandom returns a function, not a class — call it directly
const rng = seedrandom(roomSeed);
function nextWord(): Word {
  return wordBank[Math.floor(rng() * wordBank.length)];
}
```

> ⚠️ CROSS-CHECK: `seedrandom` is a real npm package ([davidbau/seedrandom](https://github.com/davidbau/seedrandom)) — the usage above is correct. The seed should be a string (e.g., room ID + match timestamp) to guarantee uniqueness per match.

#### What the server actually needs to sync (minimal):
- Room state (players, seed, start timestamp)
- Player scores / combo (for display only)
- Player elimination events

> ⚠️ CORRECTION: The original said "server authoritative state" for word positions. That's wrong for this game type — you don't need the server to track word Y positions. Each client runs the same deterministic simulation. The server only needs to sync events (typed word, missed word, eliminated). This cuts server load dramatically and eliminates the latency problem.

> ⚠️ CORRECTION: "Last player standing" with a shared word stream has a fairness problem — if Player A is faster, they clear words before Player B can type them. Fix: each player has their own independent word stream (same seed, different lane). Competition is score-based, not word-stealing.

---

## 5. 🧱 Tech Stack

### Frontend
- **Rendering**: Canvas 2D API — sufficient for this game. PixiJS (WebGL) shines when you have hundreds of animated sprites, but for falling text it adds overhead (WebGL context setup, texture atlases for fonts) without meaningful gain. Canvas 2D `fillText` is fast enough for ~50 words on screen. If you later add particle effects or animated backgrounds, revisit PixiJS.
- **Framework**: Plain JS or React for UI shell + Canvas for game. Don't render the game inside React's DOM — keep them separate.
- **State**: Zustand for UI state. Game state lives in the game loop, not in React.
- **Input**: Raw `keydown` events on `window`, not on DOM elements. Prevents focus issues.

> ⚠️ CORRECTION: The original suggested Redux. Redux is overkill here and adds unnecessary boilerplate. Zustand or even a simple event emitter is the right call for game state that changes 60x/second.

### Backend
- **Runtime**: Node.js with Fastify (faster than Express, better TypeScript support)
- **WebSocket**: `ws` library directly, or Socket.IO if you want rooms/namespaces out of the box
- **Auth**: JWT (stateless) — no sessions needed for a game

### Database
- **PostgreSQL**: users, word banks, leaderboards, word stats
- **Redis**: active game sessions, pub/sub for multiplayer room events, rate limiting

---

## 6. 📊 Analytics & Telemetry

Track these events (send async, never block the game loop):

```ts
type GameEvent =
  | { type: 'word_typed';   word: string; reactionMs: number; correct: boolean }
  | { type: 'word_escaped'; word: string; }
  | { type: 'session_end';  wpm: number; accuracy: number; score: number; mode: string }
  | { type: 'rage_quit';    timeIntoSession: number; lastDifficulty: number }
```

> ➕ ADDITION: `rage_quit` is the most valuable event. If players consistently quit at difficulty level 3.2 after 4 minutes, that's your balancing target. Track it explicitly.

> ➕ ADDITION: Funnel analysis — track `session_start → first_word_typed → 1min_survived → 5min_survived → session_end`. Drop-off at each stage tells you exactly where to focus.

---

## 7. 🎨 UI/UX

Critical details the original glossed over:

- **Font choice matters more than anything else.** Use a monospace font (JetBrains Mono, Fira Code) — characters have equal width, which makes words visually stable as they fall and easier to read at speed.
- **Typed feedback**: highlight already-typed characters in the falling word (green prefix). This is standard in games like TypeRacer and dramatically reduces frustration.
- **Word color by difficulty**: subtle hue shift (white → yellow → orange → red) gives players instant visual priority cues.
- **Canvas DPI**: multiply canvas dimensions by `window.devicePixelRatio` and scale the context — otherwise text looks blurry on retina displays.

```js
canvas.width = width * devicePixelRatio;
canvas.height = height * devicePixelRatio;
ctx.scale(devicePixelRatio, devicePixelRatio);
```

---

## 8. 🚀 MVP Scope

### Phase 1 — Playable Core (1–2 weeks)
- Canvas renderer with lane system
- Falling words with deltaTime-based movement
- Input buffer with prefix matching + word locking
- Basic scoring
- Loss condition

### Phase 2 — Feel (1 week)
- Adaptive difficulty (smoothed score)
- Combo multiplier
- Typed character highlighting
- Screen shake + danger zone

### Phase 3 — Depth (2 weeks)
- Game modes (Survival, Time Attack, Zen)
- Word categories + learning layer
- Spaced repetition for weak words
- Local stats persistence

### Phase 4 — Scale (2–3 weeks)
- Auth + profiles
- Global leaderboard
- Multiplayer (deterministic, score-based)

---

## 9. ⚠️ Hard Problems (with solutions)

| Problem | Solution |
|---|---|
| Input feel | Buffer reset on no-match, word locking on single prefix match |
| 144hz vs 60hz parity | Always use `deltaTime` — never px/frame |
| First-frame deltaTime spike | Init `lastTime = null`, skip frame 1, cap deltaTime at 100ms |
| Tab-resume teleport | Cap deltaTime at 100ms — rAF delivers huge elapsed time on resume |
| Difficulty oscillation | Smoothed difficulty score (EMA), rolling metric windows |
| Word overlap | Lane system on spawn |
| Retina blur | Scale canvas by `devicePixelRatio` |
| Multiplayer fairness | Independent streams per player, score-based competition |
| Multiplayer sync cost | Deterministic simulation, server only syncs events |
| Rage quit | Danger mode pause mechanic, "last word" slowdown |
| SM-2 ease hell | Floor ease_factor at 1.3, use full Wozniak formula not just the multiplier |

---

## 🔑 Final Perspective

The original architecture is a solid starting point but has several implementation-level mistakes that would cause real bugs: per-frame speed (breaks on high refresh rate monitors), no lane system (words overlap), Redux for game state (wrong tool), server-authoritative word positions in multiplayer (unnecessary and expensive), and treating spaced repetition as optional when it's the core of the learning layer.

Cross-check caught additional bugs introduced in the first revision: `lastTime = 0` causes a massive deltaTime spike on frame 1 (rAF timestamp is not 0 — it's ms since page load); the SM-2 formula was oversimplified to just `interval * ease_factor`, missing the bootstrapping phase and the ease factor floor of 1.3; and the `normalize` helper was called without being defined.

The real opportunity here is the **learning layer + adaptive difficulty combo**. No mainstream typing game does both well. If you nail the feel of the input (buffer reset, word locking, character highlighting) and the difficulty curve feels fair, you have something genuinely worth building.

Ship Phase 1 fast. The game loop and input matching are where you'll spend 80% of your debugging time — everything else is features on top of a working core.
