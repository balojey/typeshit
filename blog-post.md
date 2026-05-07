# I Built a 3D Space Typing Game Where Every Word You Type Launches a Missile

#BuiltWithMeDo

---

Typing practice is boring. Let's be honest - staring at a paragraph of text and watching a WPM counter tick up isn't exactly compelling gameplay. So I built something different.

**TypeSh!t** is a 3D space defense typing game. Enemy ships chase you through deep space and launch missiles labeled with words. Type the word correctly, and your ship fires an interceptor that blows it up. Miss too many, and your ship goes down.

It sounds simple. It's not.

## The idea

I wanted to answer one question: *what if getting better at typing felt like winning a dogfight?*

Most typing tools treat practice as a chore. But the skills that make someone a fast typist - pattern recognition, rapid decision-making, muscle memory - are the same skills gamers already train for hours. The gap isn't motivation. It's packaging.

So I wrapped a spaced repetition learning engine inside a Three.js space combat game. You're not studying vocabulary. You're defending your ship. The words you struggle with keep coming back (that's SM-2 doing its job), but you don't notice because you're too busy trying to survive.

## What's under the hood

The game runs entirely in the browser:

- **Three.js + @react-three/fiber** for the 3D scene - PBR materials on the ships, particle systems for engine trails and explosions, volumetric fog for depth, dynamic point lights that follow every vessel
- **274,000+ words** from the word-list npm package, categorized by length into five difficulty tiers
- **Adaptive difficulty** that uses an exponential moving average across your WPM, accuracy, and reaction time - no sudden spikes, just a curve that tightens as you improve
- **Seven game modes** - Survival, Time Attack, Zen, Boss Mode, Hardcore, Speedrun, and Story Mode with AI-narrated short stories where the narrative itself becomes your missiles
- **Real-time multiplayer** using deterministic simulation - both clients run the same seeded word sequence independently, so the server only syncs scores and events, not positions
- **A full tournament system** - single and double elimination brackets, match lobbies with ready-state sync, shared seeds for competitive fairness
- **Supabase** for persistence - leaderboards, word stats, replays, friend lists, tournament data, all cross-device

## The hard parts

**Making it feel right.** The difference between a frustrating typing game and an addictive one comes down to three tiny decisions: reset the input buffer immediately when no word matches (don't make players press Escape), lock to a single word once only one candidate remains, and slow down the last missile by 20% when it's close to hitting you. That last one is a trick stolen from Tetris - perceived fairness matters more than actual fairness.

**3D text readability.** Word labels are canvas-rendered textures applied to billboard sprites. Getting them readable against a dark space background with explosions and particle trails everywhere required semi-transparent dark backing panels and careful color-coding by difficulty (white → yellow → orange → red → magenta).

**Multiplayer without lag.** Each client runs its own simulation from a shared seed. The server never tracks missile positions. This means zero interpolation, zero rubber-banding, and the game feels identical whether you're playing solo or in a tournament match.

**One leaderboard entry per player per mode.** Sounds simple until you're handling concurrent submissions, UPSERT logic, rank recalculation, and cache invalidation - all inside a database transaction that needs to complete before the Game Over screen renders.

## The modes

- **Survival** - enemies spawn faster until you can't keep up
- **Time Attack** - 60 seconds, score multiplier 1.5x, survive or don't
- **Zen** - no death, just flow state with capped difficulty
- **Boss Mode** - armored missiles that take three correct typings to destroy
- **Hardcore** - one hit and you're done, 2x multiplier
- **Speedrun** - fixed 50-word sequence, same for everyone, ranked by completion time
- **Story Mode** - an AI narrator tells a short story, and the words from the narrative appear as missiles synchronized with the narration

## What I'd build next

Sound design is the obvious gap - explosions should boom, engines should hum, and that last-missile-approaching moment needs a heartbeat. Ship customization (unlockable skins earned through play), spectator mode for tournaments, and mobile touch controls are all on the roadmap.

The bigger vision is B2B: companies licensing focus sessions for domain-specific typing training. A medical team practicing anatomy terms. A trading desk drilling financial vocabulary. Same game, different word banks, real skill development.

## Try it

TypeSh!t is live and playable. Defend your ship. Get faster. Watch your weak words disappear from the rotation as SM-2 pushes them further and further apart.

Every word you nail is a missile that never hits you again.

---

#BuiltWithMeDo
