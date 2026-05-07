# TypeSh!t - Space Defense Typing Game

## Inspiration

We noticed that typing practice tools are either boring (plain text on a white screen) or gamified in the most shallow way possible (a progress bar and some confetti). Meanwhile, actual gamers spend hours in high-intensity environments that demand fast reflexes and pattern recognition - the exact same skills that make someone a fast typist. We asked: what if typing practice felt like piloting a spaceship under attack? What if every word you typed launched a missile that saved your life?

The other spark was vocabulary building. Spaced repetition works, but flashcard apps feel like homework. We wanted to hide the learning loop inside something people would actually want to play - a 3D space combat game where the words you struggle with keep coming back until you nail them.

## What it does

TypeSh!t drops you into a 3D space environment rendered with Three.js. Enemy ships pursue you and launch missiles labeled with words. You type the word correctly to fire an interceptor that destroys the incoming threat. Miss too many and your ship goes down.

The game features seven modes - Survival, Time Attack, Zen, Boss Mode, Hardcore, Speedrun, and Story Mode (with AI-narrated short stories where the narrative text becomes your missiles). It includes real-time multiplayer space battles with deterministic simulation, a full tournament system with brackets and match lobbies, global leaderboards with one entry per player per mode, spaced repetition that surfaces your weak words more frequently, and a 274,000+ word dictionary from the word-list package that ensures you never see the same sequence twice.

Players can exit at any time with their score saved, compete in organized tournaments, challenge friends, create custom word banks, track detailed analytics, and watch replays of their sessions - all persisted to a Supabase database for cross-device access.

## How we built it

- **Frontend**: React + TypeScript with Vite, Three.js and @react-three/fiber for the 3D scene, Zustand for state management
- **3D Rendering**: PBR materials, particle systems for engine trails and explosions, volumetric fog, dynamic point lights attached to ships, billboard text sprites for word labels
- **Game Engine**: requestAnimationFrame loop with deltaTime-based movement, 8-lane system for enemy positioning, bounding sphere collision detection, adaptive difficulty with exponential moving average smoothing
- **Backend**: Node.js + Fastify with WebSocket support for multiplayer, JWT authentication, PostgreSQL via Supabase for persistence
- **Multiplayer**: Deterministic simulation using shared seeds (seedrandom) - the server syncs events, not positions, keeping latency invisible
- **Learning Layer**: SM-2 spaced repetition algorithm tracking per-word stats, with due words weighted 3x in spawn selection
- **Word Source**: word-list npm package (274k+ English words) categorized by length into difficulty tiers and filtered by category
- **Tournament System**: Single and double elimination brackets, shared seeds for fair matches, match lobbies with ready-state synchronization

## Challenges we ran into

**3D performance with text rendering** - Rendering word labels as 3D sprites that always face the camera while maintaining readability against a dark space background required careful texture generation and contrast management. We solved it with canvas-rendered text applied to sprite materials with semi-transparent dark backing panels.

**Deterministic multiplayer fairness** - Both players need identical missile sequences without the server tracking positions. Getting seedrandom to produce consistent results across clients while handling edge cases (disconnects, forfeits, tournament standardization) took significant iteration.

**Adaptive difficulty that doesn't oscillate** - A naive approach makes the game flip between too easy and too hard every few seconds. The exponential moving average with a 10% step rate and rolling metric windows (last 10 words for WPM, last 20 keystrokes for accuracy) finally gave us a curve that feels fair.

**Score saving on exit** - Ensuring scores persist whether the player finishes naturally, exits via the button, or closes the tab required coordinating navigator.sendBeacon, immediate database writes, and leaderboard UPSERT logic with unique constraints per player per mode.

**Leaderboard atomicity** - Enforcing one entry per player per mode with proper rank calculation required database transactions, UPSERT logic, and careful cache invalidation to keep the leaderboard responsive while maintaining data integrity.

## Accomplishments that we're proud of

- A full 3D space combat environment running at 60 FPS with particle effects, dynamic lighting, and PBR materials - all driven by typing input
- Seven distinct game modes that each feel meaningfully different, including AI-narrated Story Mode
- A tournament system with bracket generation, match lobbies, and competitive fairness guarantees
- Spaced repetition that actually works invisibly - players get better at their weak words without realizing they're studying
- Real-time multiplayer that feels responsive despite being fully deterministic on each client
- A leaderboard system that handles concurrent submissions, rank recalculation, and one-entry-per-player enforcement atomically
- The word-list integration giving players access to 274,000+ words, making every session feel fresh

## What we learned

- **Game feel is everything** - The buffer reset on no-match, word locking on single prefix, and the "last word tension" slowdown make the difference between frustrating and addictive. These tiny UX decisions took more iteration than any backend feature.
- **Deterministic simulation is powerful** - By having each client run the same seeded simulation independently, we eliminated the need for server-authoritative game state and made multiplayer work with minimal bandwidth.
- **3D adds immersion but costs complexity** - Three.js gives us stunning visuals, but managing object disposal, texture memory, WebGL context loss, and frame rate monitoring added an entire layer of engineering that a 2D canvas game wouldn't need.
- **Spaced repetition needs the full algorithm** - Our first attempt oversimplified SM-2 to just `interval * ease_factor`. The bootstrapping phase (1 day, then 6 days) and the ease factor floor at 1.3 are critical for words to actually space out correctly.
- **Database transactions aren't optional for leaderboards** - Without atomic UPSERT operations and proper unique constraints, concurrent score submissions create duplicate entries and incorrect rankings.

## What's next for TypeSh!t

- **Sound design** - Background music, explosion effects, engine hums, and a heartbeat sound during last-word tension
- **Ship customization** - Unlockable ship skins, missile effects, and engine trail colors earned through gameplay
- **Spectator mode** - Watch live tournament matches and multiplayer battles
- **Mobile touch controls** - Swipe-to-type interface for mobile players
- **Team battles** - 2v2 and 4v4 cooperative multiplayer modes
- **Procedural stories** - AI-generated narratives that adapt to player skill level and vocabulary
- **Multi-language support** - Word lists in Spanish, French, German, and Japanese for language learners
- **B2B focus sessions** - Enterprise licensing for companies wanting domain-specific typing training (medical, legal, finance)
- **Advanced anti-cheat** - Server-side session validation and score verification for competitive integrity
- **VR mode** - Immersive cockpit experience using WebXR
