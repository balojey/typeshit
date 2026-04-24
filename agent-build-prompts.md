# Agent Build Prompts — Falling Word Typing Game

Each prompt below is standalone and self-contained. Feed them to an AI agent in order.
They map directly to the phased architecture in `typesh!t.md`.

---

## PROMPT 1 — Project Scaffold

Set up a monorepo for a browser-based falling-word typing game. The structure should be:

```
/
├── client/   (Vite + React + TypeScript)
├── server/   (Node.js + Fastify + TypeScript)
└── shared/   (shared TypeScript types used by both client and server)
```

For the client:
- Initialize with Vite using the React + TypeScript template
- Install: `zustand`, `uuid`, `@types/uuid`
- Create a blank `index.html` that mounts a React root
- Create `client/src/main.tsx` that renders `<App />`
- Create `client/src/App.tsx` that renders a full-screen `<canvas id="game-canvas">` element alongside a `<div id="hud">` overlay. Keep them as siblings, not nested. The canvas must NOT be managed by React state.

For the server:
- Initialize a Node.js project with TypeScript
- Install: `fastify`, `@fastify/websocket`, `dotenv`
- Create `server/src/index.ts` that starts a Fastify server on port 3001 with a `/health` GET route returning `{ status: 'ok' }`

For shared:
- Create `shared/types.ts` and define the following TypeScript interfaces exactly as specified:

```ts
export interface Word {
  id: string;
  text: string;
  x: number;
  y: number;
  speed: number;        // px per second — NOT px per frame
  lane: number;         // column index, prevents overlap
  difficulty: 1 | 2 | 3 | 4 | 5;
  category: string;
  spawnedAt: number;    // DOMHighResTimeStamp
  isMatched: boolean;
  tensionApplied?: boolean; // set true when last-word speed reduction has been applied
}

export interface GameState {
  words: Word[];
  score: number;
  combo: number;
  lives: number;
  isRunning: boolean;
  mode: string;
}

export interface PlayerMetrics {
  wpm: number;
  accuracy: number;
  reactionMs: number;
  missRate: number;
}

export interface DifficultyParams {
  spawnInterval: number;   // ms
  fallSpeed: number;       // px/sec
  wordLengthMin: number;
  wordLengthMax: number;
  maxActiveWords: number;
}
```

Do not add any game logic yet. Just scaffold, install dependencies, and verify both `client` and `server` compile without errors.

---

## PROMPT 2 — Game Loop + Canvas Renderer

Inside `client/src/`, create the core game engine. Do not use React state for any game data — all game state lives in the game loop module.

Create `client/src/engine/gameLoop.ts`:
- Export a `startGameLoop(canvas: HTMLCanvasElement)` function
- Use `requestAnimationFrame`, NOT `setInterval`
- Initialize `lastTime` to `null` — skip the first frame entirely (do not calculate deltaTime on frame 0)
- Cap deltaTime at `0.1` seconds (100ms) to prevent word teleporting when the tab regains focus
- The loop must execute in this exact order each frame:
  1. Calculate deltaTime
  2. Update word Y positions: `word.y += word.speed * deltaTime`
  3. Check loss condition: mark words as escaped if `word.y > canvasHeight`
  4. Remove matched and escaped words from the active list
  5. Spawn new words if spawn timer has elapsed
  6. Render the frame

Create `client/src/engine/renderer.ts`:
- Import `Word`, `GameState` from `shared/types.ts` — do not redeclare them locally
- Export a `render(ctx: CanvasRenderingContext2D, state: GameState, canvasWidth: number, canvasHeight: number)` function — the canvas dimensions are passed in, not read from the canvas element inside render
- Use font `"18px 'JetBrains Mono', 'Fira Code', monospace"` for all word text
- For each word, draw the text at `(word.x, word.y)`
- Color words by difficulty: difficulty 1 = white, 2 = yellow, 3 = orange, 4 = red, 5 = magenta
- Clear the canvas with a dark background (`#0d0d0d`) each frame before drawing

Note: DPI scaling (`devicePixelRatio`) is handled by a separate `initCanvas(canvas)` / `resizeCanvas(canvas)` function — do NOT put it inside `render()`. The render function only draws; it never touches canvas dimensions. The full DPI setup is specified in Prompt 15.

Wire `startGameLoop` into `App.tsx` using a `useEffect` with an empty dependency array. Pass the canvas ref. Do not call it more than once.

---

## PROMPT 3 — Lane System + Word Spawner

Create `client/src/engine/spawner.ts`.

The spawner is responsible for creating `Word` objects and assigning them lanes to prevent overlap.

Requirements:
- Divide the canvas into `N_LANES = 8` equal-width columns
- Track which lanes are currently occupied (a lane is occupied if a word is actively falling in it)
- On spawn, pick a random unoccupied lane. If all lanes are occupied, do not spawn
- Assign `word.x` as the center of the chosen lane
- Pull words from a static word bank defined in `client/src/data/wordBank.ts` — create this file with at least 100 words across 3 categories: `'general'`, `'tech'`, `'finance'`. Each entry should have `{ text, category, difficulty }`.
- Set `word.speed` to a base value of `80` px/sec (this will be overridden by the difficulty engine later)
- Set `word.y` to `-20` (just above the canvas top)
- Set `word.spawnedAt` to the current `performance.now()` timestamp
- Export a `spawnWord(canvasWidth: number, activeWords: Word[]): Word | null` function

Integrate the spawner into the game loop from Prompt 2. Spawn a new word every 2000ms by default. Use a `timeSinceLastSpawn` accumulator updated with `deltaTime`, not `setInterval`.

---

## PROMPT 4 — Input Handler + Buffer Logic

Create `client/src/engine/inputHandler.ts`.

This module handles all keyboard input for the game. Attach a single `keydown` listener to `window` (not to any DOM element — this prevents focus issues).

Implement the following logic exactly:

```ts
let buffer = '';
let lockedWordId: string | null = null;

function onKeyDown(e: KeyboardEvent, activeWords: Word[], onMatch: (word: Word) => void) {
  if (e.key.length !== 1) return; // ignore Shift, Enter, Arrow keys, etc.
  // Note: space is also filtered here — this is intentional. All words in the word bank
  // are single words with no spaces. Multi-word phrase support is a future concern.

  buffer += e.key;

  // Scope search to locked word if one exists
  const wordsToSearch = lockedWordId
    ? activeWords.filter(w => w.id === lockedWordId)
    : activeWords;

  // Check for exact match within scoped list
  const exactMatch = wordsToSearch.find(w => w.text === buffer);
  if (exactMatch) {
    onMatch(exactMatch);
    buffer = '';
    lockedWordId = null;
    return;
  }

  // Check for partial matches — also scoped to locked word if lock exists
  const partials = wordsToSearch.filter(w => w.text.startsWith(buffer));

  if (partials.length === 0) {
    buffer = '';       // no word starts with this — reset immediately
    lockedWordId = null;
    return;
  }

  if (partials.length === 1) {
    lockedWordId = partials[0].id; // lock to this word
  }
}
```

Export:
- `initInputHandler(getActiveWords: () => Word[], onMatch: (word: Word) => void): () => void` — returns a cleanup function that removes the event listener
- `getBuffer(): string` — returns current buffer for rendering the typed prefix highlight
- `getLockedWordId(): string | null`

In the renderer, use `getBuffer()` and `getLockedWordId()` to draw the already-typed prefix of the locked/matching word in green, and the remaining characters in the word's difficulty color.

---

## PROMPT 5 — Scoring + Combo System

Create `client/src/engine/scoring.ts`.

Implement the scoring system with the following exact rules:

```
base_word_score = word.text.length * 10 * word.difficulty
combo_multiplier = Math.min(1 + (combo * 0.1), 4.0)
score += Math.floor(base_word_score * combo_multiplier)

combo increments by 1 on every successful word match
combo resets to 0 when:
  - a word escapes the screen (reaches the bottom untyped)
  - a buffer reset occurs due to no partial match (a "miss")
```

Export:
- `ScoreState { score: number, combo: number, comboMultiplier: number }`
- `onWordMatched(word: Word, state: ScoreState): ScoreState`
- `onWordEscaped(state: ScoreState): ScoreState`
- `onMiss(state: ScoreState): ScoreState`

Also implement lives tracking:
- Player starts with 3 lives
- Each word that escapes costs 1 life
- When lives reach 0, call the `onGameOver` callback

Export an `initScoring(onGameOver: () => void)` function that sets up the module and returns `{ onWordMatched, onWordEscaped, onMiss, getState }`. The `onGameOver` callback is passed in at init time — do not use a global.

Create `client/src/components/HUD.tsx` — a React component rendered in `<div id="hud">` (overlaid on the canvas using CSS `position: absolute`). It should display: current score, combo multiplier (hide it when combo is 0), lives remaining as icons, and current WPM. Use Zustand to share score state between the game loop and the HUD. Do not pass game state through React props or context — the game loop writes to the Zustand store directly.

---

## PROMPT 6 — Adaptive Difficulty Engine

Create `client/src/engine/difficultyEngine.ts`.

The difficulty engine reads rolling player metrics and outputs spawn parameters. It must NOT jump difficulty instantly — use an exponential moving average (EMA) to smooth transitions.

Import `PlayerMetrics` and `DifficultyParams` from `shared/types.ts` — do not redeclare them locally.

Implement exactly:

```ts
function normalize(value: number, min: number, max: number): number {
  return Math.max(0, Math.min(1, (value - min) / (max - min)));
}

function updateDifficultyScore(metrics: PlayerMetrics, currentDifficulty: number): number {
  const wpmScore   = normalize(metrics.wpm, 20, 120);
  const accScore   = normalize(metrics.accuracy, 0.5, 1.0);
  const reactScore = normalize(metrics.reactionMs, 3000, 300); // lower ms = higher score

  const raw = (wpmScore * 0.4) + (accScore * 0.4) + (reactScore * 0.2);
  return currentDifficulty + (raw - currentDifficulty) * 0.1; // EMA, 10% step
}
```

Map the difficulty score (0–1) to spawn parameters:
- `spawnInterval`: lerp from 3000ms (score=0) to 800ms (score=1)
- `fallSpeed`: lerp from 60 px/sec (score=0) to 220 px/sec (score=1)
- `wordLengthMin`: lerp from 3 (score=0) to 8 (score=1), floored
- `wordLengthMax`: lerp from 5 (score=0) to 14 (score=1), floored
- `maxActiveWords`: lerp from 3 (score=0) to 10 (score=1), floored

Track rolling metrics:
- WPM: rolling average of last 10 matched words (use `word.text.length` and reaction time)
- Accuracy: rolling window of last 20 keystrokes (correct = keystroke that advanced a partial match, incorrect = keystroke that caused a buffer reset)
- ReactionMs: time from `word.spawnedAt` to first correct keystroke on that word

Update difficulty score after every matched or escaped word. Export `getDifficultyParams(): DifficultyParams` for the spawner and game loop to consume.

---

## PROMPT 7 — Danger Zone + Game Feel

Add the pressure system and game feel layer to the existing engine.

**Danger Mode:**
- Define `DANGER_THRESHOLD = 6` (active words on screen simultaneously)
- When `activeWords.length >= DANGER_THRESHOLD`, set `dangerMode = true` and pause spawning (do not spawn new words until count drops below threshold)
- While in danger mode, apply a red vignette overlay on the canvas: draw a radial gradient from transparent center to `rgba(255, 0, 0, 0.25)` at the edges
- Exit danger mode when `activeWords.length < DANGER_THRESHOLD - 1` (hysteresis — prevents rapid toggling)

**Last Word Tension:**
- When `activeWords.length === 1` and the remaining word's `y > canvasHeight * 0.65`, reduce that word's speed by 20% (multiply by 0.8, apply once, not every frame — use a flag on the word object `tensionApplied: boolean`)

**Screen Shake:**
- When a word escapes (life lost), trigger a screen shake: for 300ms, offset the canvas context by a random value between -6 and 6 pixels on both axes each frame
- Implement this using `ctx.save()` before translating and `ctx.restore()` after the frame renders — do NOT manually reverse the translate, as floating point drift will accumulate over time:
  ```js
  ctx.save();
  ctx.translate(shakeX, shakeY);
  // ... draw everything ...
  ctx.restore();
  ```

**Typed Character Highlight:**
- This should already be partially in place from Prompt 4. Confirm the renderer draws the typed prefix in `#00ff88` (bright green) and the remaining characters in the word's difficulty color. The prefix length comes from `getBuffer().length` when the word is locked.

All of these are purely client-side. No server changes needed.

---

## PROMPT 8 — Game Modes System

Create `client/src/engine/modes/` directory with the following files.

First, define the mode interface in `client/src/engine/modes/types.ts`:

```ts
export interface GameMode {
  id: string;
  name: string;
  spawnStrategy: (state: GameState, params: DifficultyParams) => Word | null;
  winCondition: (state: GameState) => boolean;
  lossCondition: (state: GameState) => boolean;
  scoreMultiplier: number;
  modifiers: ('no_backspace' | 'timed' | 'lives' | 'no_hints')[];
}
```

Implement these three modes as separate files. Boss Mode, Hardcore, and Speedrun are intentionally deferred — do not implement them yet, but leave a comment in `modeRegistry.ts` noting they are planned.

`survival.ts` — Words fall progressively faster. No win condition. Loss = 0 lives. `scoreMultiplier: 1.0`. Uses adaptive difficulty normally.

`timeAttack.ts` — 60-second timer. Win condition = timer reaches 0 AND lives > 0. Loss = lives reach 0 before timer ends. `scoreMultiplier: 1.5`. Add a countdown timer to the HUD when this mode is active.

`zen.ts` — No loss condition (lossCondition always returns false). Lives are cosmetic only. Adaptive difficulty still runs but caps fall speed at 120 px/sec. `scoreMultiplier: 0.5`. No screen shake.

Create a `modeRegistry.ts` that exports `getModeById(id: string): GameMode` and `getAllModes(): GameMode[]`.

In `App.tsx`, add a mode selection screen that appears before the game starts. It should show the three mode cards with name and a one-line description. Clicking a card starts the game in that mode. Style it minimally — dark background, centered cards, monospace font.

---

## PROMPT 9 — Learning Layer + Word Stats

Create `client/src/learning/wordStats.ts`.

Track per-word performance and implement SM-2 spaced repetition to surface weak words more frequently.

**Word stat tracking:**

```ts
interface WordStat {
  word: string;
  attempts: number;
  misses: number;
  escapes: number;
  avgReactionMs: number;
  lastSeen: number;
  smInterval: number;      // days until next review
  smRepetitions: number;   // SM-2 repetition count
  smEaseFactor: number;    // SM-2 ease factor, min 1.3, starts at 2.5
}
```

After each word interaction, update its stat and run the SM-2 algorithm:

```
q = quality score:
  - typed correctly AND reactionMs < 1000: q = 5
  - typed correctly AND reactionMs < 2500: q = 3
  - missed (buffer reset) or escaped:      q = 1

SM-2 update:
  if q >= 3:
    if repetitions == 0: interval = 1
    elif repetitions == 1: interval = 6
    else: interval = round(last_interval * ease_factor)
    repetitions += 1
  else:
    repetitions = 0
    interval = 1

  ease_factor = max(1.3, ease_factor + 0.1 - (5 - q) * (0.08 + (5 - q) * 0.02))
```

Persist all word stats to `localStorage` under the key `'typeshit_word_stats'`. Load on game start, save after each session.

Modify the spawner to use word stats when selecting words: give a 3x weight to words whose `lastSeen + smInterval * 86400000 <= Date.now()` (i.e., due for review). If no words are due, fall back to random selection from the word bank filtered by current difficulty word length range.

---

## PROMPT 10 — Analytics + Telemetry

Create `client/src/analytics/telemetry.ts`.

Implement a lightweight, non-blocking analytics module. All events are queued and flushed asynchronously — they must NEVER block or delay the game loop.

Define the event types:

```ts
type GameEvent =
  | { type: 'session_start';   mode: string; timestamp: number }
  | { type: 'word_typed';      word: string; reactionMs: number; correct: boolean }
  | { type: 'word_escaped';    word: string }
  | { type: 'first_word';      timestamp: number }
  | { type: 'milestone';       label: '1min' | '5min'; timestamp: number }
  | { type: 'session_end';     wpm: number; accuracy: number; score: number; mode: string; durationMs: number }
  | { type: 'rage_quit';       timeIntoSessionMs: number; lastDifficulty: number }
```

Implementation rules:
- Maintain an in-memory event queue (array)
- Export `track(event: GameEvent): void` — pushes to queue, never awaits
- Flush the queue every 5 seconds using `setInterval`, sending a POST to `/api/events` as a JSON batch. If the server is unreachable, silently discard (do not retry, do not throw)
- On `session_end` or `rage_quit`, flush immediately (don't wait for the interval)
- Track `rage_quit` by listening for the browser's `beforeunload` event if the game is still running (isRunning === true). Fire the event synchronously using `navigator.sendBeacon` to guarantee delivery on page close.
- Fire `session_start` immediately when the game loop begins (mode is known at that point)
- Fire `first_word` on the first successful word match of the session
- Fire `milestone` with label `'1min'` when the session clock passes 60 seconds, and `'5min'` when it passes 300 seconds — track elapsed session time in the game loop using accumulated deltaTime

On the server side, create `server/src/routes/events.ts` — a POST `/api/events` route that accepts an array of `GameEvent` objects, logs them to console for now (no DB yet), and returns `{ received: n }`.

---

## PROMPT 11 — Backend: Auth + User Profiles

Set up authentication and user profiles on the server.

Install on server: `@fastify/jwt`, `@fastify/cookie`, `bcrypt`, `@types/bcrypt`, `pg`, `@types/pg` (node-postgres)

Create `server/src/db/schema.sql` with the following tables:

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(32) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE word_stats (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  word TEXT NOT NULL,
  attempts INT DEFAULT 0,
  misses INT DEFAULT 0,
  escapes INT DEFAULT 0,
  avg_reaction_ms FLOAT DEFAULT 0,
  sm_interval INT DEFAULT 1,
  sm_repetitions INT DEFAULT 0,
  sm_ease_factor FLOAT DEFAULT 2.5,
  last_seen TIMESTAMPTZ,
  UNIQUE(user_id, word)
);

CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  mode VARCHAR(32),
  score INT,
  wpm FLOAT,
  accuracy FLOAT,
  duration_ms INT,
  played_at TIMESTAMPTZ DEFAULT NOW()
);
```

Create these routes in `server/src/routes/auth.ts`:
- `POST /api/auth/register` — accepts `{ username, email, password }`, hashes password with bcrypt (cost 12), inserts user, returns JWT
- `POST /api/auth/login` — accepts `{ email, password }`, verifies hash, returns JWT
- `GET /api/auth/me` — protected route, returns user profile from JWT

JWT secret must come from `process.env.JWT_SECRET`. Tokens expire in 7 days.

On the client, create `client/src/api/auth.ts` with `register()`, `login()`, and `getMe()` functions using `fetch`. Store the JWT in `localStorage` under `'typeshit_token'`. Create a minimal login/register UI in `client/src/components/AuthScreen.tsx` — shown before the mode select screen if no token exists.

On successful login, sync word stats from localStorage to the server: read `'typeshit_word_stats'` from localStorage and POST each entry to a new route `POST /api/word-stats/sync` (create this route — it upserts into the `word_stats` table using `ON CONFLICT (user_id, word) DO UPDATE`). On login, also pull the user's server-side word stats and merge them into localStorage (server wins on conflict, using the higher `attempts` count as the tiebreaker).

---

## PROMPT 12 — Leaderboard

Create a global leaderboard backed by PostgreSQL.

Add to `server/src/db/schema.sql`:

```sql
CREATE TABLE leaderboard (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  username VARCHAR(32) NOT NULL,
  mode VARCHAR(32) NOT NULL,
  score INT NOT NULL,
  wpm FLOAT NOT NULL,
  accuracy FLOAT NOT NULL,
  achieved_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_leaderboard_mode_score ON leaderboard(mode, score DESC);
```

Create `server/src/routes/leaderboard.ts`:
- `POST /api/leaderboard` — protected (JWT required). Accepts `{ mode, score, wpm, accuracy }`. Inserts a new entry. Returns the entry with rank.
- `GET /api/leaderboard/:mode` — public. Returns top 50 entries for the given mode, ordered by score DESC. Include `rank` as a computed field.

On the client, create `client/src/components/Leaderboard.tsx` — shown on the game over screen. Displays the top 10 for the current mode. Highlight the current player's row if they appear in the list. Show their rank even if outside top 10 (fetch separately). Use a simple table layout with monospace font.

Submit the score automatically on `session_end` if the user is authenticated.

Note on Speedrun mode (to be implemented later): when Speedrun mode is added, its leaderboard entries are directly comparable because all players type the same fixed word list. The leaderboard query for mode `'speedrun'` should additionally return the fixed word list hash used, so clients can verify parity.

---

## PROMPT 13 — Multiplayer: Room + Matchmaking

Implement real-time multiplayer using WebSockets. The architecture is deterministic — the server does NOT track word positions. Each client runs the same simulation using a shared seed.

On the client, install `seedrandom` and its types: `npm install seedrandom @types/seedrandom`

On the server, create `server/src/multiplayer/roomManager.ts`:

```ts
interface Room {
  id: string;
  seed: string;           // roomId + startTimestamp, used by both clients
  players: Player[];
  status: 'waiting' | 'countdown' | 'active' | 'finished';
  startTimestamp: number | null;
}

interface Player {
  id: string;
  username: string;
  score: number;
  combo: number;
  isEliminated: boolean;
  ws: WebSocket;
}
```

WebSocket message types the server handles:
- `join_room` — add player to a waiting room (create one if none available). Respond with `room_joined { roomId, seed, players }`
- `score_update` — player sends `{ score, combo }` every 2 seconds. Server broadcasts to all other players in the room.
- `word_matched` — player sends `{ wordId }`. Server broadcasts to others (for visual effects only).
- `player_eliminated` — player sends when they hit 0 lives. Server broadcasts, checks if only one player remains → send `game_over { winnerId }`.

On the client, create `client/src/net/wsClient.ts`:
- Connects to `ws://localhost:3001/ws`
- Exports `joinRoom()`, `sendScoreUpdate(score, combo)`, `sendWordMatched(wordId)`, `sendEliminated()`
- Exposes an event emitter for incoming messages: `on('room_joined', ...)`, `on('score_update', ...)`, `on('game_over', ...)`

In the game, when multiplayer mode is active:
- Use the seed from `room_joined` to initialize `seedrandom` for word selection
- Each player has their own independent word stream — do NOT share words between players
- Show opponent score/combo in a sidebar HUD panel (updated from `score_update` events)

---

## PROMPT 14 — Word Bank Expansion + Category Focus Sessions

Expand the word bank and add category-based focus sessions.

In `client/src/data/wordBank.ts`, expand to at least 500 words across these categories:
- `general` — common English words, 3–12 chars
- `tech` — programming terms, CLI commands, framework names, acronyms
- `finance` — financial terms, trading vocabulary, economic concepts
- `medical` — anatomy, conditions, procedures (for future B2B use)

Each word entry: `{ text: string, category: string, difficulty: 1 | 2 | 3 | 4 | 5 }`. Difficulty should reflect word length AND typing complexity (e.g., words with rare letter combos like 'zx', 'qw' are harder than their length suggests).

Add a "Focus Session" mode to the mode selection screen:
- Player picks a category before starting
- The spawner exclusively draws from that category during the session
- Session is 5 minutes, Zen-mode rules (no loss condition), but tracks and displays per-category accuracy at the end
- Show a post-session summary: words practiced, top 5 hardest words (by miss rate), WPM for this category vs overall average

This is the B2B monetization hook — the UX should feel like a deliberate training tool, not just a game mode.

---

## PROMPT 15 — Polish, DPI, Font Loading, and Final Wiring

This is the final integration pass. Fix all known rough edges before the game is considered shippable.

**Canvas DPI fix** — confirm this is applied on every canvas resize, not just on init:
```js
function resizeCanvas(canvas: HTMLCanvasElement) {
  const dpr = window.devicePixelRatio || 1;
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width * dpr;
  canvas.height = rect.height * dpr;
  const ctx = canvas.getContext('2d')!;
  ctx.scale(dpr, dpr);
}
window.addEventListener('resize', () => resizeCanvas(canvas));
```

**Font loading** — before starting the game loop, wait for the monospace font to load:
```js
await document.fonts.load('18px "JetBrains Mono"');
```
If it fails or times out after 2 seconds, fall back to `'Courier New'`. Do not start the game loop until the font is ready — unloaded fonts cause a flash of wrong-width text on the first few frames.

**Input edge cases** — verify these work correctly:
- Backspace does nothing (buffer does not shrink — this is intentional, not a bug)
- Caps Lock does not break matching (normalize input to lowercase, normalize word bank to lowercase)
- Rapid typing does not cause double-registration (the `keydown` handler must be idempotent per event)

**Game over screen** — when lives reach 0, stop the game loop, show a game over overlay with: final score, WPM, accuracy, combo peak, and buttons for "Play Again" (same mode) and "Change Mode". If authenticated, show leaderboard rank.

**Pause on tab hide** — rAF already pauses when the tab is hidden, but reset `lastTime = null` on `visibilitychange` so the deltaTime cap kicks in cleanly when the tab is restored:
```js
document.addEventListener('visibilitychange', () => {
  if (document.hidden) lastTime = null;
});
```

**Verify end-to-end**: start a Survival session, type 10 words, let 3 escape, confirm score/combo/lives update correctly, confirm difficulty parameters shift after 10 words, confirm word stats are saved to localStorage on session end.
