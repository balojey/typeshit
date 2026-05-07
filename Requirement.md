# Requirements Document

## 1. Application Overview

### 1.1 Application Name
Typesh!t — Space Defense Typing Game

### 1.2 Application Description
A browser-based 3D space-themed typing game where players pilot a spaceship through deep space while being pursued by multiple enemy ships. Enemy vessels launch missile attacks labeled with words. Players defend their ship by typing words correctly, which fires interceptor missiles to destroy incoming threats. The game features adaptive difficulty, multiple game modes including a new Story Mode with AI narration, real-time multiplayer space battles, tournament system, spaced repetition learning, and comprehensive analytics. All application data is persisted to database for cross-device access and historical analysis. The game utilizes the word-list package to provide an extensive vocabulary library, offering virtually infinite word variety and creating an immersive vocabulary-building experience for gamers. The entire game environment is rendered in 3D using Three.js and @react-three/fiber, providing realistic space visuals, volumetric lighting, particle effects, and physically-based rendering. A fully functional leaderboard system tracks and displays player rankings across all game modes, with each player appearing only once per mode showing their highest score. Tournament functionality enables competitive organized play with brackets, scheduling, and prize tracking. Player scores are saved even when the game is exited early, ensuring progress is never lost.

### 1.3 Reference Files
1. Architecture document: agent-build-prompts.md
2. Technical specification: typesh!t.md

## 2. Users and Use Cases

### 2.1 Target Users
- Casual gamers seeking typing practice with engaging 3D space combat visuals
- Competitive players aiming for leaderboard rankings and tournament victories
- Professional users requiring category-specific typing training
- Multiplayer enthusiasts looking for real-time competitive typing challenges
- Tournament organizers hosting competitive typing events
- Esports participants competing in structured tournaments
- Vocabulary learners seeking to expand their word knowledge through gameplay
- Players seeking immersive 3D space experience
- Story enthusiasts who enjoy narrative-driven typing experiences

### 2.2 Core Use Cases
- Pilot spaceship through 3D space while defending against enemy missile attacks
- Type words correctly to launch interceptor missiles
- Practice typing speed and accuracy through 3D space combat gameplay
- Experience AI-narrated stories while typing in Story Mode
- Compete on global leaderboards across different game modes with highest score per player
- Register for and participate in tournaments
- Create and manage tournaments as organizer
- Track tournament brackets and standings
- View tournament schedules and match results
- Compete in tournament matches with standardized rules
- View personal ranking and compare with top players
- Track leaderboard position changes over time
- Train on specific vocabulary categories
- Challenge friends in real-time multiplayer 3D space battles
- Track personal progress and identify weak words through spaced repetition
- Add friends and view their progress
- Create custom multiplayer lobbies with invite codes
- Record and share gameplay sessions
- Share scores and achievements on social platforms
- Create and share custom word lists
- Receive AI-driven suggestions for skill improvement
- View detailed analytics on typing improvement over time
- Exit game session at any time during gameplay with score saved
- Encounter diverse vocabulary from extensive dictionary word library
- Experience realistic 3D space environment with volumetric effects

## 3. Page Structure and Functionality

### 3.1 Overall Structure
```
Root
├── Authentication Screen
├── Mode Selection Screen
├── Tournament Hub Screen
├── Tournament Creation Screen
├── Tournament Details Screen
├── Tournament Bracket View
├── Tournament Match Lobby
├── Story Selection Screen
├── 3D Game Scene (with HUD Overlay and Exit Button)
├── Game Over Screen
├── Leaderboard View
├── Friends Management Screen
├── Private Room Creation Screen
├── Replay Viewer Screen
├── Social Sharing Screen
├── Custom Word Bank Manager
├── Learning Path Dashboard
└── Progress Reports Screen
```

### 3.2 Authentication Screen

**Purpose**: User registration and login

**Layout**:
- Dark space-themed background with animated 3D star field
- Centered form with glowing borders
- Monospace font styling
- Scrollable container to accommodate all form elements

**Functionality**:
- Register form: username (max 32 chars), email, password input fields
- Login form: email and password input fields
- Submit button triggers authentication API call
- On successful authentication:
  - Store JWT token in localStorage under key typeshit_token
  - Sync local word stats to server via POST /api/word-stats/sync
  - Pull server-side word stats and merge into localStorage (server wins on conflict, using higher attempts count as tiebreaker)
  - Navigate to Mode Selection Screen
- Display error messages for failed authentication
- User account data stored in database: userId, username, email, passwordHash, createdAt, lastLoginAt
- Entire screen scrollable when content exceeds viewport height

**Visual Contrast Requirements**:
- Form input fields: white text (#FFFFFF) on dark background (#1a1a2e)
- Form labels: light gray text (#e0e0e0)
- Submit button: bright cyan (#00d9ff) background with dark text (#0a0a12)
- Error messages: bright red text (#ff4444)
- Form borders: glowing cyan (#00d9ff) with 2px width

### 3.3 Mode Selection Screen

**Purpose**: Choose game mode before starting

**Layout**:
- 3D animated space background with rotating planets and nebulae
- Centered mode cards with holographic effect
- Seven mode cards displayed: Survival, Time Attack, Zen, Boss Mode, Hardcore, Speedrun, Story Mode
- Each card shows mode name and one-line description
- Focus Session option with category picker
- Navigation buttons to Tournament Hub, Leaderboard, Friends, Private Rooms, Replays, Custom Word Banks, Learning Path, Progress Reports
- Scrollable container to display all mode cards and navigation options

**Functionality**:
- Click mode card to start game in selected mode
- Story Mode card navigates to Story Selection Screen
- Focus Session mode:
  - Display category picker: general, tech, finance, medical, custom categories
  - Player selects category before starting
  - Session duration: 5 minutes
  - Zen-mode rules apply (no loss condition)
- After mode selection, load game with chosen configuration
- User mode preferences stored in database for quick access
- Tournament Hub button navigates to Tournament Hub Screen
- Leaderboard button navigates to Leaderboard View
- Entire screen scrollable when content exceeds viewport height

**Visual Contrast Requirements**:
- Mode card background: semi-transparent dark (#1a1a2e with 80% opacity)
- Mode card title: bright white text (#FFFFFF) with 24px font size
- Mode card description: light gray text (#c0c0c0) with 14px font size
- Mode card border: glowing cyan (#00d9ff) on hover
- Navigation buttons: bright cyan background (#00d9ff) with dark text (#0a0a12)
- Category picker options: white text (#FFFFFF) on dark background (#1a1a2e)

**Mode Definitions**:
- **Survival**: Enemy ships spawn progressively faster, no win condition, loss when ship health reaches 0, score multiplier 1.0
- **Time Attack**: 60-second timer, win if timer reaches 0 with ship health > 0, loss if ship destroyed before timer ends, score multiplier 1.5
- **Zen**: No loss condition, ship health cosmetic only, enemy spawn rate capped, score multiplier 0.5, no camera shake
- **Boss Mode**: Periodic boss ship encounters with armored missiles requiring multiple hits, score multiplier 1.8
- **Hardcore**: Permadeath variant with single hit point, score multiplier 2.0
- **Speedrun**: Fixed missile sequence with leaderboard for fastest completion time, score multiplier 1.2
- **Story Mode**: AI-narrated short story (2-5 minutes) with typing gameplay, score multiplier 1.3

### 3.4 Story Selection Screen

**Purpose**: Choose story for Story Mode gameplay

**Layout**:
- 3D space library theme with floating holographic book displays
- Story cards displayed in grid layout
- Each card shows story title, duration, difficulty level, and brief synopsis
- Filter options: duration (2-3 min, 3-4 min, 4-5 min), difficulty (easy, medium, hard), genre (sci-fi, adventure, mystery, fantasy)
- Back button to return to Mode Selection Screen
- Scrollable container to display all story cards

**Visual Contrast Requirements**:
- Story card background: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Story title: bright white text (#FFFFFF) with 20px font
- Story synopsis: light gray text (#c0c0c0) with 14px font
- Duration indicator: bright yellow text (#ffff00) with 16px font
- Difficulty badge: color-coded (green for easy, yellow for medium, red for hard)
- Genre tag: bright cyan text (#00d9ff) with dark background
- Play button: bright green background (#00ff00) with dark text (#0a0a12)
- Filter controls: white text (#FFFFFF) on dark background (#1a1a2e)
- Back button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Display list of available stories from database
- Filter stories by duration, difficulty, and genre
- Click story card to view full synopsis and metadata
- Play button starts Story Mode game with selected story
- Story metadata includes: storyId, title, duration, difficulty, genre, synopsis, narratorVoice, wordCount, createdAt
- Stories stored in database with full narrative text and timing data
- Track completed stories per user in database
- Display completion indicator on previously completed stories
- Entire screen scrollable when content exceeds viewport height

### 3.5 Tournament Hub Screen

**Purpose**: Central hub for tournament discovery, registration, and management

**Layout**:
- 3D space station command center theme
- Tournament list with filtering options
- Create Tournament button (for authorized users)
- My Tournaments section showing registered and created tournaments
- Featured tournaments carousel at top
- Tournament status indicators
- Scrollable container to display all tournaments

**Visual Contrast Requirements**:
- Tournament cards: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Tournament title: bright white text (#FFFFFF) with 22px font
- Tournament status badges: color-coded (green for open, yellow for in-progress, red for completed)
- Create Tournament button: bright green background (#00ff00) with dark text (#0a0a12)
- Filter controls: white text (#FFFFFF) on dark background (#1a1a2e)
- Featured carousel: bright borders with glowing effect
- Registration button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Display list of all active and upcoming tournaments
- Filter tournaments by: status (open, in-progress, completed), game mode, entry fee, prize pool, start date
- Search tournaments by name or organizer
- Click tournament card to view Tournament Details Screen
- Register for tournament via quick registration button
- Create Tournament button navigates to Tournament Creation Screen
- My Tournaments section shows:
  - Tournaments user is registered for
  - Tournaments user created (if organizer)
  - Current tournament standings
- Featured tournaments displayed in carousel with auto-rotation
- Real-time updates for tournament status changes
- Tournament data loaded from database
- Entire screen scrollable when content exceeds viewport height

### 3.6 Tournament Creation Screen

**Purpose**: Create and configure new tournaments

**Layout**:
- 3D holographic interface theme
- Tournament configuration form
- Bracket preview panel
- Prize distribution settings
- Scrollable container to display all configuration options

**Visual Contrast Requirements**:
- Form sections: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Section titles: bright white text (#FFFFFF) with 20px font
- Input fields: white text (#FFFFFF) on dark background (#0a0a12)
- Input labels: light gray text (#c0c0c0)
- Bracket preview: white lines on dark background with player slots
- Create button: bright green background (#00ff00) with dark text (#0a0a12)
- Cancel button: bright red background (#ff3333) with white text (#FFFFFF)

**Functionality**:
- Tournament configuration fields:
  - Tournament name (required, max 64 chars)
  - Description (optional, max 500 chars)
  - Game mode (dropdown: Survival, Time Attack, Boss Mode, Hardcore, Speedrun, Story Mode)
  - Bracket type (dropdown: Single Elimination, Double Elimination)
  - Maximum participants (dropdown: 8, 16, 32, 64)
  - Registration deadline (date/time picker)
  - Tournament start date (date/time picker)
  - Entry fee (optional, numeric input)
  - Prize pool distribution (percentage sliders for 1st, 2nd, 3rd places)
  - Match duration (minutes, for Time Attack mode)
  - Story selection (for Story Mode tournaments)
  - Custom rules (optional text area)
- Bracket preview updates dynamically based on participant count and bracket type
- Validate all required fields before submission
- Submit tournament creation via POST /api/tournaments/create
- On success, navigate to Tournament Details Screen for created tournament
- Tournament data persisted to database
- Entire screen scrollable when content exceeds viewport height

### 3.7 Tournament Details Screen

**Purpose**: Display comprehensive tournament information and manage participation

**Layout**:
- 3D tournament arena theme
- Tournament header with title and status
- Information panel with tournament details
- Participant list with registration status
- Match schedule section
- Bracket visualization
- Chat or announcement section
- Scrollable container to display all tournament information

**Visual Contrast Requirements**:
- Header background: dark gradient (#1a1a2e to #0a0a12) with bright cyan border (#00d9ff)
- Tournament title: bright white text (#FFFFFF) with 28px font
- Status badge: color-coded (green, yellow, red) with white text
- Information labels: light gray text (#c0c0c0)
- Information values: white text (#FFFFFF) with 18px font
- Participant list: white text (#FFFFFF) on dark background
- Registered participants: bright green checkmark (#00ff00)
- Match schedule: white text with time in bright cyan (#00d9ff)
- Register button: bright green background (#00ff00) with dark text (#0a0a12)
- Unregister button: bright red background (#ff3333) with white text (#FFFFFF)
- Start Tournament button (organizer only): bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Display tournament information:
  - Tournament name and description
  - Game mode and bracket type
  - Current participant count / maximum participants
  - Registration deadline
  - Tournament start date
  - Entry fee and prize pool
  - Organizer username
  - Tournament status
  - Story title (if Story Mode tournament)
- Participant list shows all registered players with usernames
- Registration button allows user to join tournament
- Unregister button allows user to leave tournament (before start)
- Match schedule displays upcoming matches with date/time
- Bracket visualization shows tournament structure and match results
- Chat/announcement section for organizer updates
- Start Tournament button (organizer only) begins tournament and locks registration
- Real-time updates for participant changes and match results
- Navigate to Tournament Match Lobby when match is ready
- Tournament data loaded from database
- Entire screen scrollable when content exceeds viewport height

### 3.8 Tournament Bracket View

**Purpose**: Visualize tournament bracket structure and match progression

**Layout**:
- Full-width bracket visualization
- Hierarchical tree structure for elimination brackets
- Match nodes with player names and scores
- Scrollable and zoomable canvas

**Visual Contrast Requirements**:
- Bracket lines: bright cyan (#00d9ff) connecting match nodes
- Match nodes: dark background (#1a1a2e) with bright borders
- Player names: white text (#FFFFFF) with 16px font
- Scores: bright yellow text (#ffff00) with 18px bold font
- Winner highlight: bright green border (#00ff00)
- Current match highlight: pulsing cyan border (#00d9ff)
- Completed matches: dimmed with gray overlay
- Upcoming matches: bright borders

**Functionality**:
- Display bracket structure based on tournament type:
  - Single Elimination: standard bracket tree
  - Double Elimination: winners bracket and losers bracket
- Each match node shows:
  - Player 1 username and score
  - Player 2 username and score
  - Match status (scheduled, in-progress, completed)
  - Match start time
- Click match node to view match details
- Highlight current user's matches
- Highlight active matches in progress
- Auto-scroll to current user's next match
- Zoom controls for large brackets
- Pan/drag to navigate bracket
- Real-time updates as matches complete
- Bracket data loaded from database
- Responsive layout for different screen sizes

### 3.9 Tournament Match Lobby

**Purpose**: Pre-match waiting room for tournament participants

**Layout**:
- 3D space hangar theme
- Player readiness indicators
- Match information panel
- Countdown timer
- Chat section
- Scrollable container for chat messages

**Visual Contrast Requirements**:
- Player cards: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Player usernames: bright white text (#FFFFFF) with 20px font
- Ready status: bright green checkmark (#00ff00) with Ready text
- Not ready status: gray indicator (#666666) with Not Ready text
- Match info panel: dark background (#0a0a12) with white text (#FFFFFF)
- Countdown timer: bright yellow text (#ffff00) with 36px bold font
- Chat messages: white text (#FFFFFF) on dark background
- Ready button: bright green background (#00ff00) with dark text (#0a0a12)
- Cancel button: bright red background (#ff3333) with white text (#FFFFFF)

**Functionality**:
- Display match information:
  - Tournament name
  - Match round (e.g., Round 1, Quarterfinals, Finals)
  - Game mode
  - Match rules
  - Story title (if Story Mode tournament)
- Show both players with readiness status
- Ready button toggles player ready state
- Match starts automatically when both players ready
- Countdown timer (30 seconds) before match start
- Chat section for player communication
- Cancel button returns to Tournament Details Screen (forfeits match)
- Real-time synchronization of ready states
- Navigate to 3D Game Scene when match starts
- Match lobby data stored in database
- Chat messages scrollable when exceeding viewport height

### 3.10 3D Game Scene and HUD

**3D Scene Setup**:
- Full-screen Three.js scene rendered using @react-three/fiber
- PerspectiveCamera with FOV 75, positioned at (0, 5, 15) looking toward origin
- Scene background: deep space skybox with high-resolution star field texture
- Ambient lighting with low intensity (0.2) for base illumination
- Directional light simulating distant star, positioned at (10, 10, 10) with intensity 0.8
- Point lights attached to player ship and enemy ships for dynamic lighting
- Volumetric fog effect with dark blue color (#0a0a1e) for depth perception

**3D Space Environment**:
- Animated 3D star field with particle system (5000 particles)
- Stars rendered as point sprites with varying sizes and brightness
- Stars drift slowly in Z-axis to create parallax depth effect
- Distant nebula clouds rendered as volumetric sprites with color gradients (purple, blue, cyan)
- Rotating planet models in far background for environmental detail
- Asteroid field particles drifting through scene
- Dynamic lighting from distant stars creating realistic space ambiance

**Player Spaceship 3D Model**:
- Positioned at (0, -3, 0) in 3D space (bottom center of viewport)
- 3D model: sleek fighter-class spacecraft with detailed geometry
- Hull material: metallic blue with physically-based rendering (PBR)
- Glowing cyan engine thrusters with particle emitters (#00d9ff)
- Animated engine flame particles with heat distortion effect
- Cockpit with emissive glass material
- Wing-mounted weapon ports with glowing indicators
- Point light attached to ship (cyan #00d9ff, intensity 2.0) for local illumination
- Ship scale: 1.5 units width, 2.0 units length
- Subtle idle animation: gentle bobbing motion on Y-axis

**Enemy Spaceship 3D Models**:
- Spawn at Z = -30 (far distance) and move toward camera (Z increases)
- 3D model: hostile interceptor design with angular geometry
- Hull material: metallic red with battle damage decals (#ff3333)
- Glowing red engine thrusters with particle emitters
- Weapon ports with red emissive material
- Point light attached to each ship (red #ff3333, intensity 1.5)
- Ship scale: 1.2 units width, 1.5 units length
- Multiple enemy ships visible simultaneously in 3D space
- Each ship assigned to lane position (X-axis) with slight random offset
- Ships rotate slowly on Y-axis for dynamic appearance

**Missile 3D Models**:
- Enemy missiles: 3D cylindrical projectile with red glowing trail
- Missile geometry: elongated cylinder with pointed nose cone
- Material: metallic with red emissive stripes (#ff3333)
- Particle trail: red glowing particles emitted from rear
- Trail particles fade over distance with alpha decay
- Missiles travel from enemy ship position toward player ship (Z-axis movement)
- Missiles rotate on longitudinal axis during flight
- Scale: 0.3 units diameter, 1.0 units length

**Interceptor Missile 3D Models**:
- Launched from player ship on correct word match
- 3D model: sleek projectile with cyan glowing core
- Material: metallic with cyan emissive bands (#00d9ff)
- Particle trail: bright cyan particles with sparkle effect
- Missiles travel from player ship toward specific target enemy missile
- Homing behavior: missiles adjust trajectory to track moving targets
- Scale: 0.25 units diameter, 0.8 units length
- Speed: 15 units per second in 3D space

**Word Label 3D Rendering**:
- Word labels rendered as 3D text sprites using THREE.Sprite
- Positioned above missiles in 3D space with billboard behavior (always face camera)
- Font: 18px bold JetBrains Mono, Fira Code, or monospace fallback
- Text rendered to canvas texture, applied to sprite material
- Color-coded difficulty with high contrast:
  - Difficulty 1: bright white (#FFFFFF)
  - Difficulty 2: bright yellow (#ffff00)
  - Difficulty 3: bright orange (#ff9900)
  - Difficulty 4: bright red (#ff3333)
  - Difficulty 5: bright magenta (#ff00ff)
- Typed prefix highlighted in bright green (#00ff88) with dark background
- Remaining characters in difficulty color with dark background
- Text background: semi-transparent dark panel (#0a0a12 with 85% opacity)
- Padding: 4px around text for readability
- Glow effect: emissive material property for text visibility in dark space

**Story Mode Narrative Display**:
- AI narrator voice plays audio narration of story
- Narrative text displayed in HUD overlay panel (top center)
- Text panel: dark semi-transparent background (#000000 with 85% opacity)
- Narrative text: bright white (#FFFFFF) with 18px font
- Current sentence highlighted in bright cyan (#00d9ff)
- Text scrolls automatically as narration progresses
- Words from narrative appear as missiles for typing
- Missiles spawn synchronized with narrative pacing
- Story progress bar displayed below narrative panel
- Progress bar: bright green fill (#00ff00) on dark gray background (#333333)

**Explosion 3D Effects**:
- Triggered on missile intercept collision
- Particle explosion system with 200 particles
- Particles emit in spherical pattern from collision point
- Particle colors: gradient from bright yellow to orange to red
- Particles have velocity decay and gravity effect
- Bright flash light at explosion center (intensity 5.0, duration 0.2s)
- Shockwave ring effect expanding from center
- Explosion duration: 1.5 seconds with particle fade-out

**Ship Damage 3D Effects**:
- Player ship flashes bright red material on hit
- Spark particle emitters activate at impact point
- Sparks: bright yellow particles with short lifespan
- Hull damage decals appear on ship surface
- Smoke particle emitter activates when health below 50%
- Ship model shows progressive damage (cracks, burn marks) as health decreases
- Camera shake effect on hit (disabled in Zen mode and Story Mode)

**HUD Overlay** (HTML overlay on 3D scene):
- Current score display (top left): bright white text (#FFFFFF) with 24px font, dark semi-transparent background
- Combo multiplier with glow effect (top left, below score): bright cyan text (#00d9ff) with 20px font
- Ship health bar with segments (bottom center): bright green (#00ff00) fill, dark gray (#333333) background, white (#FFFFFF) border
- Current WPM (top right): bright white text (#FFFFFF) with 20px font, dark semi-transparent background
- Countdown timer (Time Attack mode only, top center): bright yellow text (#ffff00) with 28px font
- Completion timer (Speedrun mode only, top center): bright cyan text (#00d9ff) with 28px font
- Story progress bar (Story Mode only, below narrative panel): bright green (#00ff00) fill, dark gray (#333333) background
- Narrative text panel (Story Mode only, top center): white text (#FFFFFF) on dark semi-transparent background
- Boss ship health bar (Boss Mode only, top center): bright red (#ff3333) fill, dark gray (#333333) background
- Opponent ship health/score sidebar (Multiplayer mode only, right side): white text (#FFFFFF) on dark semi-transparent background
- Tournament match info (Tournament mode only, top center): white text (#FFFFFF) showing round and opponent
- Recording indicator (when replay recording active, top right corner): bright red circle (#ff0000) with pulsing animation
- Active missile count indicator (bottom right): bright orange text (#ff9900) with 18px font
- Exit button (top right corner): bright red background (#ff3333) with white text (#FFFFFF), labeled Exit or X icon, 40px x 40px

**Exit Button Functionality**:
- Position: top right corner of HUD overlay
- Visual: bright red background (#ff3333) with white X icon or Exit text (#FFFFFF)
- Size: 40px x 40px minimum clickable area
- On click:
  - Pause game loop and 3D rendering
  - Stop AI narration audio (Story Mode)
  - Save current session data to database immediately
  - Submit current score to leaderboard via POST /api/leaderboard/submit
  - Display confirmation dialog: Your progress has been saved. Are you sure you want to exit?
  - Confirmation dialog buttons: Yes (bright red #ff3333) and No (bright cyan #00d9ff)
  - If Yes: stop game loop, dispose 3D scene resources, navigate to appropriate screen (Mode Selection or Tournament Details)
  - If No: resume game, resume narration (Story Mode), close dialog
- Hover effect: brighter red background (#ff5555)
- Tournament matches: exiting forfeits match and awards win to opponent

**Game Loop Functionality**:
- Runs on requestAnimationFrame via Three.js render loop
- Frame execution order:
  1. Calculate deltaTime (capped at 0.1 seconds)
  2. Update camera position and rotation (if dynamic camera enabled)
  3. Update star particle positions for parallax effect
  4. Update enemy ship 3D positions (Z-axis movement toward camera)
  5. Update enemy ship rotations and animations
  6. Update missile 3D positions: missile.position.z += missile.speed * deltaTime
  7. Update interceptor missile positions with homing behavior toward targets
  8. Update particle systems (engine trails, explosions, sparks)
  9. Update AI narration audio playback (Story Mode)
  10. Update narrative text display and story progress (Story Mode)
  11. Check collision: interceptor hits enemy missile (3D distance check)
  12. Check loss condition: enemy missile reaches player ship (Z-position threshold)
  13. Remove destroyed missiles and completed interceptors from scene
  14. Spawn new enemy ships and missiles if spawn timer elapsed
  15. Update lighting (point lights follow ships)
  16. Render 3D scene with Three.js renderer
  17. Record frame data if replay recording active
- First frame skipped (lastTime initialized to null)
- Pause on tab hide, reset lastTime on visibilitychange

**Lane System in 3D Space**:
- Scene divided into 8 lanes along X-axis
- Lane positions: X = -7, -5, -3, -1, 1, 3, 5, 7
- Each enemy ship assigned to unoccupied lane on spawn
- Enemy ship spawns at Z = -30 (far distance)
- Enemy ships move forward (Z increases) at constant speed
- Enemy ships drift slightly left/right within lane boundaries (X-axis)
- Missiles launched from enemy ship 3D position
- If all lanes occupied, skip spawn

**Missile Spawning in 3D**:
- Base spawn interval: 2000ms (adjusted by difficulty engine)
- Missiles pulled from word-list package dictionary or custom word banks
- Story Mode: missiles pulled from narrative text in sequence
- Each missile has: text, category, difficulty (1-5)
- Initial missile.speed: 4 units/sec in Z-axis (adjusted by difficulty)
- Missile spawns at enemy ship 3D position
- Missile inherits enemy ship lane X-position
- missile.spawnedAt set to performance.now() timestamp
- Speedrun mode uses fixed missile sequence
- Tournament matches use shared seed for deterministic missile sequence
- Story Mode uses narrative-synchronized missile sequence

**Input Handling**:
- Single keydown listener on window
- Ignore non-character keys (Shift, Enter, Arrow keys, Space)
- Buffer accumulates typed characters
- Exact match triggers interceptor launch toward specific target missile
- Partial match locks to missile if only one candidate
- No partial match resets buffer immediately
- Backspace does nothing (buffer does not shrink)
- Input normalized to lowercase
- Caps Lock does not break matching

**Interceptor Missile 3D Mechanics**:
- On successful word match, interceptor missile spawned from player ship
- Interceptor starts at player ship 3D position (0, -3, 0)
- Interceptor travels toward specific target enemy missile in 3D space
- Homing behavior: interceptor adjusts trajectory each frame to track target
- Speed: 15 units/sec with acceleration curve
- Collision detection: 3D distance between interceptor and target < 1.5 units
- On collision:
  - Both missiles removed from scene
  - 3D explosion effect at collision point
  - Score awarded
  - Combo incremented
- Interceptor removed if target already destroyed

**Danger Mode Visual Effects**:
- Triggers when activeMissiles.length >= 6
- Pause spawning until count drops below 5
- Red alert overlay: pulsing red post-processing effect
- Increase ambient red lighting intensity
- Warning klaxon sound effect (optional)
- Camera subtle shake effect

**Critical Threat Indicator**:
- When any missile reaches Z > 10 (close to player)
- Missile label pulses bright red (#ff0000) with increased scale
- Missile trail particle intensity increases
- Missile emissive material brightness increases
- Reduce missile speed by 20% (multiply by 0.8, apply once using tensionApplied flag)

**Camera Shake Effect**:
- Triggered when missile hits player ship (health lost)
- Duration: 300ms
- Random camera position offset between -0.3 and 0.3 units on X and Y axes
- Camera returns to original position with smooth interpolation
- Disabled in Zen mode and Story Mode

**Ship Damage Visual in 3D**:
- Player ship material flashes bright red on hit
- Spark particle system emits from impact point
- Hull damage decals applied to ship mesh
- Smoke particle emitter activates when health low
- Ship model shows progressive damage geometry

### 3.11 Game Over Screen

**Purpose**: Display session results and navigation options

**Layout**:
- Overlay on 3D scene (scene paused in background)
- Centered content with dark semi-transparent background (#000000 with 85% opacity)
- 3D explosion animation in background (if ship destroyed)
- Scrollable container to display all session results and options

**Displayed Information**:
- Final score: bright white text (#FFFFFF) with 32px font
- WPM (words per minute): bright cyan text (#00d9ff) with 24px font
- Accuracy percentage: bright green text (#00ff00) with 24px font
- Peak combo: bright yellow text (#ffff00) with 20px font
- Missiles intercepted count: bright white text (#FFFFFF) with 18px font
- Missiles hit count: bright red text (#ff3333) with 18px font
- Story completion percentage (Story Mode only): bright cyan text (#00d9ff) with 24px font
- Completion time (Speedrun mode only): bright cyan text (#00d9ff) with 24px font
- Tournament match result (Tournament mode only): Victory or Defeat with bright colors
- Leaderboard rank (if authenticated): bright orange text (#ff9900) with 20px font
- Personal best indicator if new high score achieved
- Rank change indicator (up/down arrows with position change)
- Top 10 leaderboard for current mode: white text (#FFFFFF) on dark background
- Player's row highlighted with bright cyan background (#00d9ff with 20% opacity)

**Functionality**:
- Play Again button: bright cyan background (#00d9ff) with dark text (#0a0a12), restart with same mode
- Change Mode button: bright blue background (#0099ff) with dark text (#0a0a12), return to Mode Selection Screen
- View Full Leaderboard button: bright purple background (#9900ff) with white text (#FFFFFF), navigate to Leaderboard View
- Share Score button: bright green background (#00ff00) with dark text (#0a0a12), navigate to Social Sharing Screen
- Save Replay button: bright purple background (#9900ff) with white text (#FFFFFF), save current session replay
- Return to Tournament button (Tournament mode only): bright cyan background (#00d9ff) with dark text (#0a0a12), navigate to Tournament Details Screen
- Return to Story Selection button (Story Mode only): bright cyan background (#00d9ff) with dark text (#0a0a12), navigate to Story Selection Screen
- **Critical: Automatic score submission to leaderboard must execute immediately on game over, before displaying results**
- **Critical: Score submission must complete successfully and receive confirmation from server before displaying rank**
- Display rank change animation if position improved
- Focus Session mode shows additional summary
- Speedrun mode shows completion time and rank by time
- Story Mode shows completion percentage and narrative summary
- Tournament mode submits match result to server
- Session data persisted to database
- Entire screen scrollable when content exceeds viewport height

### 3.12 Leaderboard View

**Purpose**: Display global rankings across all game modes with unique player entries per mode

**Layout**:
- 3D space-themed background with animated elements
- Holographic table with glowing cyan borders (#00d9ff)
- Mode selector tabs at top: Survival, Time Attack, Zen, Boss Mode, Hardcore, Speedrun, Story Mode
- Time range filter: All Time, This Month, This Week, Today
- Monospace font
- Columns: Rank, Username, Score, WPM, Accuracy, Missiles Intercepted
- Speedrun mode columns: Rank, Username, Completion Time, Accuracy
- Story Mode columns: Rank, Username, Score, WPM, Accuracy, Story Title
- Scrollable table to display all leaderboard entries
- Pagination controls at bottom

**Visual Contrast Requirements**:
- Mode selector tabs: bright cyan background (#00d9ff) for active tab, dark background (#1a1a2e) for inactive tabs
- Tab text: dark text (#0a0a12) for active, white text (#FFFFFF) for inactive
- Time range filter: white text (#FFFFFF) on dark background (#1a1a2e) with bright cyan border (#00d9ff)
- Table header: bright white text (#FFFFFF) with 18px bold font on dark background (#1a1a2e)
- Table rows: light gray text (#c0c0c0) with 16px font on alternating dark backgrounds
- Current player row: bright cyan background (#00d9ff with 30% opacity) with white text (#FFFFFF)
- Rank numbers: bright yellow text (#ffff00)
- Top 3 ranks: gold (#ffd700), silver (#c0c0c0), bronze (#cd7f32) background highlights
- Rank change indicators: green up arrow (#00ff00) for improvement, red down arrow (#ff3333) for decline
- Pagination buttons: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Mode selector tabs switch between different game mode leaderboards
- Time range filter updates leaderboard to show rankings for selected period
- Shows top 100 entries for selected mode and time range
- **Each player appears only once per mode, displaying their highest score**
- Ordered by score descending (or completion time ascending for Speedrun)
- Highlight current player's row with bright cyan background
- Show player's rank even if outside top 100
- Display rank change indicator (up/down arrows with position change)
- Public access (no authentication required to view)
- Click username to view player profile
- Refresh button to manually update leaderboard data
- Auto-refresh every 60 seconds when viewing leaderboard
- Pagination controls to navigate through all entries
- Leaderboard data stored in database with indexes on mode, timeRange, score, userId
- Leaderboard entries cached for performance
- Real-time rank calculation based on current scores
- Entire table scrollable when content exceeds viewport height
- **Critical: Leaderboard must fetch and display data immediately on page load**
- **Critical: Loading state must display while fetching leaderboard data**
- **Critical: Error handling must catch and display any API failures with retry option**
- **Critical: Empty state must display when no scores exist with call-to-action**
- **Critical: Data validation must ensure all required fields present before rendering**

### 3.13 Friends Management Screen

**Purpose**: Manage friend connections and view friend activity

**Layout**:
- 3D space-themed background
- Friend list with avatar icons
- Search bar for finding users
- Friend request notifications section
- Friend activity feed
- Scrollable container to display all friends and activity

**Visual Contrast Requirements**:
- Search bar: white text (#FFFFFF) on dark background (#1a1a2e) with bright cyan border (#00d9ff)
- Friend list items: white text (#FFFFFF) on dark semi-transparent background
- Online status indicator: bright green circle (#00ff00)
- Offline status indicator: gray circle (#666666)
- Friend request notifications: bright orange background (#ff9900 with 20% opacity)
- Activity feed items: light gray text (#c0c0c0) on dark background (#0a0a12)
- Action buttons: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Search users by username
- Send friend request to user
- Accept or decline incoming friend requests
- Remove existing friends
- View friend list with online status indicators
- Display friend activity feed showing recent scores and achievements
- Display friend leaderboard rankings
- Display friend tournament participation
- Click friend to view detailed profile and statistics
- Challenge friend to multiplayer space battle
- Friend relationships stored in database
- Friend activity stored in database
- Entire screen scrollable when content exceeds viewport height

### 3.14 Private Room Creation Screen

**Purpose**: Create and manage custom multiplayer lobbies

**Layout**:
- 3D space station interior background
- Room configuration panel
- Invite code display with copy button
- Player list showing joined players with ship icons
- Scrollable container to display all room settings and players

**Visual Contrast Requirements**:
- Configuration panel: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Panel labels: bright white text (#FFFFFF) with 16px font
- Dropdown menus: white text (#FFFFFF) on dark background (#0a0a12)
- Invite code display: bright yellow text (#ffff00) with 24px monospace font
- Copy button: bright green background (#00ff00) with dark text (#0a0a12)
- Player list items: white text (#FFFFFF) with player ship icon
- Ready status: bright green checkmark (#00ff00)
- Not ready status: gray indicator (#666666)
- Start game button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Create new private room with custom settings
- Generate unique invite code
- Copy invite code to clipboard
- Share invite code via social platforms
- Display list of players in room with ready status
- Kick players (room creator only)
- Start game when ready (room creator only)
- Join existing room via invite code input
- Room data stored in database
- Room participants stored in database
- Entire screen scrollable when content exceeds viewport height

### 3.15 Replay Viewer Screen

**Purpose**: View and share recorded gameplay sessions

**Layout**:
- Full-screen 3D replay scene
- Playback controls overlay
- Replay information panel
- Scrollable information panel when content exceeds viewport height

**Visual Contrast Requirements**:
- Playback controls background: dark semi-transparent (#000000 with 80% opacity)
- Play/Pause button: bright cyan (#00d9ff) icon
- Speed selector: white text (#FFFFFF) on dark background (#1a1a2e)
- Timeline scrubber: bright cyan track (#00d9ff) with white handle (#FFFFFF)
- Information panel: dark background (#1a1a2e) with white text (#FFFFFF)
- Metadata labels: light gray text (#c0c0c0)
- Share button: bright green background (#00ff00) with dark text (#0a0a12)

**Functionality**:
- Load replay file from local storage or server
- Playback controls: Play/Pause, Speed adjustment, Timeline scrubber, Frame-by-frame navigation
- Display replay metadata
- Share replay: Generate shareable link, Export replay file, Post to social platforms
- Browse saved replays library
- Delete saved replays
- Replay data stored in database
- Information panel scrollable when content exceeds viewport height

### 3.16 Social Sharing Screen

**Purpose**: Share scores and achievements on social platforms

**Layout**:
- 3D space-themed background
- Score summary card with visual design
- Platform selection buttons
- Scrollable container to display all sharing options

**Visual Contrast Requirements**:
- Score card background: dark gradient (#1a1a2e to #0a0a12) with bright cyan border (#00d9ff)
- Player username: bright white text (#FFFFFF) with 28px font
- Game mode label: bright cyan text (#00d9ff) with 20px font
- Score display: bright yellow text (#ffff00) with 36px bold font
- Statistics labels: light gray text (#c0c0c0) with 14px font
- Statistics values: white text (#FFFFFF) with 18px font
- Platform buttons: brand colors with white icons
- Copy link button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Display shareable score card
- Platform sharing options: Twitter/X, Facebook, Discord, Copy link
- Generate share image with score statistics and 3D ship render
- Include replay link in share (if replay saved)
- Track share events for analytics
- Share events stored in database
- Entire screen scrollable when content exceeds viewport height

### 3.17 Custom Word Bank Manager

**Purpose**: Create, edit, and manage custom word lists

**Layout**:
- 3D space station database interface theme
- Word bank list with search
- Word bank editor panel
- Import/export controls
- Scrollable container to display all word banks and editor

**Visual Contrast Requirements**:
- Word bank list: white text (#FFFFFF) on dark background (#1a1a2e)
- Search bar: white text (#FFFFFF) with bright cyan border (#00d9ff)
- Editor panel: dark background (#0a0a12) with bright cyan borders (#00d9ff)
- Input fields: white text (#FFFFFF) on dark background (#1a1a2e)
- Word list display: monospace font, white text (#FFFFFF) with difficulty color indicators
- Add word button: bright green background (#00ff00) with dark text (#0a0a12)
- Delete word button: bright red background (#ff3333) with white text (#FFFFFF)
- Import button: bright blue background (#0099ff) with white text (#FFFFFF)
- Export button: bright purple background (#9900ff) with white text (#FFFFFF)
- Save button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Create new word bank
- Edit existing word banks
- Delete word banks
- Import word banks: CSV, JSON, Text file
- Export word banks: CSV, JSON
- Browse community-shared word banks
- Clone community word banks to personal library
- Set word bank as active for Focus Session mode
- Word bank data stored in database
- Word bank entries stored in database
- Entire screen scrollable when content exceeds viewport height

### 3.18 Learning Path Dashboard

**Purpose**: Display AI-driven recommendations for skill improvement

**Layout**:
- 3D command center interface theme
- Recommendation cards with holographic effect
- Progress visualization charts
- Skill breakdown panel
- Scrollable container to display all recommendations and charts

**Visual Contrast Requirements**:
- Recommendation cards: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Card titles: bright white text (#FFFFFF) with 20px font
- Card descriptions: light gray text (#c0c0c0) with 14px font
- Progress charts: bright colors (cyan, green, yellow) on dark background
- Skill breakdown labels: white text (#FFFFFF)
- Skill values: color-coded (green for good, yellow for average, red for needs improvement)
- Goal progress bars: bright green fill (#00ff00) on dark gray background (#333333)
- Set goal button: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Display personalized recommendations
- Show skill breakdown
- Visualize progress over time
- Set learning goals
- Track goal progress with visual indicators
- Generate weekly practice plan based on goals
- Learning goals stored in database
- Recommendations stored in database
- Entire screen scrollable when content exceeds viewport height

### 3.19 Progress Reports Screen

**Purpose**: View detailed analytics on typing improvement over time

**Layout**:
- 3D analytics dashboard with space theme
- Multiple chart panels
- Time range selector
- Export controls
- Scrollable container to display all charts and statistics

**Visual Contrast Requirements**:
- Dashboard background: dark space theme (#0a0a12)
- Chart panels: dark background (#1a1a2e) with bright cyan borders (#00d9ff)
- Chart titles: bright white text (#FFFFFF) with 18px font
- Chart lines: bright colors (cyan for WPM, green for accuracy, yellow for sessions)
- Chart axes: light gray (#c0c0c0)
- Chart labels: white text (#FFFFFF)
- Time range selector: white text (#FFFFFF) on dark background (#1a1a2e)
- Statistics cards: dark background (#1a1a2e) with bright value displays
- Export buttons: bright cyan background (#00d9ff) with dark text (#0a0a12)

**Functionality**:
- Time range selection
- Display comprehensive statistics
- Visualizations: WPM line chart, Accuracy line chart, Session frequency bar chart, Category performance radar chart, Word difficulty distribution pie chart
- Detailed word statistics
- Export reports: PDF, CSV, Share report link
- Compare performance across game modes
- View spaced repetition effectiveness metrics
- All statistics computed from database session and analytics data
- Entire screen scrollable when content exceeds viewport height

## 4. Business Rules and Logic

### 4.1 3D Rendering and Performance

**Three.js Scene Configuration**:
- WebGLRenderer with antialias enabled
- Renderer pixel ratio set to window.devicePixelRatio (capped at 2 for performance)
- Renderer output encoding: sRGBEncoding for color accuracy
- Renderer tone mapping: ACESFilmicToneMapping for realistic lighting
- Shadow mapping enabled with PCFSoftShadowMap type
- Scene fog: FogExp2 with dark blue color (#0a0a1e) and density 0.02

**Camera Setup**:
- PerspectiveCamera with FOV 75 degrees
- Aspect ratio: window.innerWidth / window.innerHeight
- Near clipping plane: 0.1 units
- Far clipping plane: 1000 units
- Camera position: (0, 5, 15) looking at origin (0, 0, 0)
- Camera updates on window resize to maintain aspect ratio

**Lighting System**:
- AmbientLight with color #ffffff and intensity 0.2 for base illumination
- DirectionalLight simulating distant star:
  - Position: (10, 10, 10)
  - Color: #ffffff
  - Intensity: 0.8
  - Cast shadows enabled
- Point lights attached to player ship and enemy ships:
  - Player ship light: cyan #00d9ff, intensity 2.0, distance 20 units
  - Enemy ship lights: red #ff3333, intensity 1.5, distance 15 units
- Dynamic lighting updates each frame as ships move

**Material System**:
- All 3D models use PBR materials (MeshStandardMaterial or MeshPhysicalMaterial)
- Metalness and roughness properties set for realistic appearance
- Emissive materials for glowing elements (engines, weapon ports, text)
- Transparent materials for cockpit glass and particle effects
- Normal maps and roughness maps for surface detail

**Particle Systems**:
- Engine trails: continuous particle emission from ship thrusters
- Explosion effects: burst particle emission on collision
- Spark effects: short-lived particles on ship damage
- Star field: large particle system for background stars
- All particle systems use PointsMaterial or custom shaders
- Particle textures loaded as sprites for visual variety

**Performance Optimization**:
- Object pooling for missiles and particles to reduce garbage collection
- Frustum culling enabled to skip rendering off-screen objects
- Level of detail (LOD) for distant objects (optional)
- Texture compression for faster loading
- Geometry instancing for repeated objects (stars, particles)
- Dispose of unused geometries, materials, and textures to free memory
- Monitor frame rate and adjust particle counts dynamically if FPS drops below 30

**3D Collision Detection**:
- Use bounding spheres for fast collision checks
- Calculate 3D distance between interceptor and target missile centers
- Collision threshold: distance < 1.5 units
- Check collisions only for active interceptors and missiles
- Spatial partitioning (optional) for large numbers of objects

**3D Asset Loading**:
- Load 3D models using GLTFLoader for player ship, enemy ships, missiles
- Load textures using TextureLoader for skybox, particles, decals
- Display loading screen with progress bar during asset loading
- Cache loaded assets for reuse across game sessions
- Handle loading errors gracefully with fallback placeholder models

### 4.2 Word-List Package Integration

**Package Usage**:
- Integrate word-list npm package as primary word source
- word-list provides extensive English dictionary words
- Package contains 274,000+ dictionary words
- Words loaded from word-list package on application initialization

**Word Loading and Categorization**:
- Load word-list dictionary into memory on app start
- Filter words by length to create difficulty tiers:
  - Difficulty 1: 3-4 characters
  - Difficulty 2: 5-6 characters
  - Difficulty 3: 7-8 characters
  - Difficulty 4: 9-11 characters
  - Difficulty 5: 12+ characters
- Create indexed data structure for fast lookup by difficulty and length
- Cache categorized words in memory for performance

**Category Assignment**:
- Assign words to categories based on common prefixes, suffixes, or word patterns
- Categories: General, Tech, Finance, Medical
- Category assignment performed during initialization
- Store category mappings in memory for fast filtering

**Word Selection Logic**:
- When spawning missile, select word from word-list dictionary
- Filter by current difficulty level word length range
- Filter by selected category (if Focus Session mode)
- Apply spaced repetition weighting (3x for due words)
- Random selection from filtered word pool
- Fallback to full dictionary if filtered pool empty
- Story Mode: extract words from narrative text in sequence

**Custom Word Banks**:
- Custom word banks take precedence over word-list dictionary when selected
- User can create custom word banks to supplement word-list words
- Custom word banks stored in database
- Word selection prioritizes custom word bank if active, otherwise uses word-list

**Performance Optimization**:
- Pre-process word-list on app initialization
- Store processed words in memory for fast access
- Use binary search or hash maps for word lookup
- Lazy load word categories on demand
- Cache frequently accessed word subsets

**Vocabulary Learning Enhancement**:
- Track unique words encountered from word-list dictionary
- Display vocabulary statistics in Progress Reports
- Highlight rare or advanced vocabulary words in game
- Provide word definitions on Game Over screen for educational value

### 4.3 Story Mode Mechanics

**Story Structure**:
- Each story contains: storyId, title, duration (2-5 minutes), difficulty, genre, synopsis, narrativeText, narratorVoice, wordCount, timingData, createdAt
- Narrative text divided into sentences with timing markers
- Each sentence contains words to be typed by player
- Stories stored in database with full text and metadata

**AI Narration System**:
- AI narrator voice synthesized using text-to-speech API
- Narrator voice options: male, female, neutral (configurable per story)
- Narration audio pre-generated and cached for performance
- Audio playback synchronized with game loop
- Narration speed adjustable (0.8x, 1.0x, 1.2x)

**Narrative Display**:
- Current sentence displayed in HUD narrative panel
- Text scrolls automatically as narration progresses
- Current sentence highlighted in bright cyan
- Completed sentences dimmed in gray
- Upcoming sentences hidden until narration reaches them

**Missile Spawning from Narrative**:
- Words extracted from current sentence in narrative
- Missiles spawn synchronized with narration pacing
- Spawn timing based on timingData in story structure
- Each word appears as missile when narrator speaks it
- Missile difficulty assigned based on word length
- Spawn rate adjusted to match narration speed

**Story Progress Tracking**:
- Progress bar displays percentage of story completed
- Progress calculated as: (current sentence index / total sentences) * 100
- Story completion tracked in database per user
- Completion percentage displayed on Game Over Screen

**Story Mode Scoring**:
- Base score calculation same as other modes
- Mode multiplier: 1.3
- Bonus points for completing story without missing words
- Completion bonus: 1000 points if 100% accuracy maintained
- Story completion percentage affects final score

**Story Mode Win/Loss Conditions**:
- No loss condition (similar to Zen mode)
- Ship health cosmetic only
- Story completes when all sentences narrated
- Player can exit early via exit button with score saved
- Partial completion tracked and displayed

**Story Mode Leaderboard**:
- Separate leaderboard for Story Mode
- Ranked by score descending
- **Each player appears only once with their highest score**
- Leaderboard columns: Rank, Username, Score, WPM, Accuracy, Story Title
- Filter by specific story or all stories
- Story completion percentage displayed in leaderboard

**Story Mode Tournament Support**:
- Tournaments can use Story Mode
- All participants play same story
- Shared seed ensures identical narration timing
- Winner determined by highest score
- Story title displayed in tournament details

**Story Mode Replay**:
- Replay includes narration audio
- Replay playback synchronizes audio with game state
- Narrative text displayed during replay
- Replay viewer shows story progress

### 4.4 Scrolling Behavior

**Global Scrolling Rules**:
- All screens except 3D Game Scene must be scrollable when content exceeds viewport height
- Scrolling implemented using CSS overflow: auto or overflow-y: scroll
- Scrollbar styling consistent with space theme: dark track with bright cyan thumb (#00d9ff)
- Smooth scrolling enabled for better user experience
- Mobile devices use native scrolling behavior

**Screen-Specific Scrolling**:
- Authentication Screen: entire form container scrollable
- Mode Selection Screen: entire content area scrollable
- Story Selection Screen: entire content area scrollable
- Tournament Hub Screen: tournament list scrollable
- Tournament Creation Screen: configuration form scrollable
- Tournament Details Screen: entire content area scrollable
- Tournament Match Lobby: chat messages scrollable
- Game Over Screen: results overlay scrollable
- Leaderboard View: table container scrollable
- Friends Management Screen: friend list and activity feed scrollable
- Private Room Creation Screen: room settings and player list scrollable
- Replay Viewer Screen: information panel scrollable
- Social Sharing Screen: entire content area scrollable
- Custom Word Bank Manager: word bank list and editor scrollable
- Learning Path Dashboard: recommendations and charts scrollable
- Progress Reports Screen: dashboard content scrollable

**3D Game Scene Scrolling**:
- 3D Game Scene itself is NOT scrollable
- Scene maintains fixed full-screen dimensions
- HUD overlay elements positioned absolutely, not affected by scrolling

### 4.5 Scoring System

**Base Score Calculation**:
```
base_word_score = word.text.length * 10 * word.difficulty
combo_multiplier = min(1 + (combo * 0.1), 4.0)
score += floor(base_word_score * combo_multiplier * mode_multiplier)
```

**Mode Multipliers**:
- Survival: 1.0
- Time Attack: 1.5
- Zen: 0.5
- Boss Mode: 1.8
- Hardcore: 2.0
- Speedrun: 1.2
- Story Mode: 1.3

**Combo Rules**:
- Increment by 1 on every successful missile intercept
- Reset to 0 when missile hits player ship or buffer reset occurs
- Story Mode: combo does not reset on missed words (Zen-like behavior)

**Ship Health System**:
- Player starts with 3 health points (Survival, Time Attack, Boss Mode, Speedrun)
- Player starts with 1 health point (Hardcore mode)
- Story Mode and Zen mode: health cosmetic only, no loss condition
- Each missile hit costs 1 health point
- Game over when health reaches 0 (except Zen mode and Story Mode)
- Hardcore mode ends immediately on first hit
- Health displayed as segmented bar in HUD

### 4.6 Boss Mode Mechanics

**Boss Ship 3D Model**:
- Boss ship appears every 20 regular missiles intercepted
- 3D model: large capital ship with detailed geometry
- Scale: 3x larger than regular enemy ships
- Position: centered at X=0, Z=-30
- Hull material: dark metallic with red accents
- Multiple weapon ports with glowing emissive materials
- Point lights attached to ship (red, high intensity)
- Boss ship has armored health bar (3 segments)

**Armored Missile 3D Model**:
- 3D model: massive missile with armored plating
- Scale: 2x larger than regular missiles
- Material: heavy metallic with damage decals
- Occupies 3 adjacent lanes visually (wide model)
- Falls at 50% normal speed
- Requires 3 successful intercepts to destroy
- Each intercept cracks armor with visual damage progression

**Armored Missile Mechanics**:
- First correct typing: armor cracks, word resets, must type again
- Second correct typing: armor breaks further, word resets, must type third time
- Third correct typing: missile destroyed, large 3D explosion animation
- Each successful typing reduces boss health bar by 1 segment
- Partial progress saved if player switches to other missiles

**Boss Defeat**:
- Boss defeated when armored missile destroyed (all 3 segments depleted)
- Award bonus score: 500 * current combo multiplier
- Combo does not reset on boss defeat
- Boss ship retreats with warp animation (3D effect)

**Boss Missile Hit**:
- If armored missile reaches player ship before destruction, lose 2 health points
- Combo resets to 0
- Larger 3D explosion animation on player ship

### 4.7 Speedrun Mode Rules

**Missile Sequence**:
- Fixed sequence of 50 missiles, same for all players
- Sequence generated from seed based on daily date
- All missiles difficulty 3-4, length 6-10 characters
- Missiles selected from word-list dictionary using seeded random

**Completion Tracking**:
- Timer starts on first keystroke
- Timer stops when all 50 missiles intercepted
- Missed missiles respawn at top (no health loss)
- Final time recorded in milliseconds

**Leaderboard Ranking**:
- Ranked by completion time (fastest first)
- **Each player appears only once with their fastest completion time**
- Ties broken by accuracy percentage

### 4.8 Adaptive Difficulty Engine

**Difficulty Score Calculation**:
```
wpmScore = normalize(metrics.wpm, 20, 120)
accScore = normalize(metrics.accuracy, 0.5, 1.0)
reactScore = normalize(metrics.reactionMs, 3000, 300)
raw = (wpmScore * 0.4) + (accScore * 0.4) + (reactScore * 0.2)
difficultyScore = currentDifficulty + (raw - currentDifficulty) * 0.1
```

**Parameter Mapping** (score 0-1):
- spawnInterval: 3000ms to 800ms
- missileSpeed: 2 units/sec to 8 units/sec (3D space)
- enemyShipSpeed: 1.5 units/sec to 4 units/sec (3D space)
- wordLengthMin: 3 to 8 chars
- wordLengthMax: 5 to 14 chars
- maxActiveMissiles: 3 to 10

**Metrics Tracking**:
- WPM: rolling average of last 10 intercepted missiles
- Accuracy: rolling window of last 20 keystrokes
- ReactionMs: time from missile.spawnedAt to first correct keystroke
- Update difficulty after every intercepted or hit missile

**Mode-Specific Adjustments**:
- Zen mode: missileSpeed and enemyShipSpeed capped at lower values
- Story Mode: adaptive difficulty disabled, spawn rate matches narration pacing
- Hardcore mode: difficulty increases 50% faster
- Speedrun mode: adaptive difficulty disabled, fixed parameters
- Tournament mode: difficulty parameters standardized for fair competition

### 4.9 Spaced Repetition (SM-2 Algorithm)

**Word Stat Tracking**:
- Per-word: attempts, misses, hits, avgReactionMs, lastSeen
- SM-2 parameters: smInterval (days), smRepetitions, smEaseFactor (min 1.3, starts 2.5)
- Word stats stored in database
- Track stats for words from both word-list dictionary and custom word banks

**Quality Score Assignment**:
- q = 5: typed correctly AND reactionMs < 1000
- q = 3: typed correctly AND reactionMs < 2500
- q = 1: missed (buffer reset) or hit player ship

**SM-2 Update Logic**:
```
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

**Word Selection Priority**:
- 3x weight to words where lastSeen + smInterval * 86400000 <= Date.now()
- Fallback to random selection from word-list dictionary filtered by current difficulty word length range
- Speedrun mode ignores spaced repetition, uses fixed sequence from word-list
- Story Mode ignores spaced repetition, uses narrative text
- Tournament mode ignores spaced repetition, uses shared seed for fairness

### 4.10 Multiplayer Logic

**Room Management**:
- Room status: waiting, countdown, active, finished
- Room seed: roomId + startTimestamp (shared by all clients)
- Maximum 2 players per room (8 players for private rooms)
- Private rooms support custom settings and invite codes
- Room data persisted to database for reconnection support

**Deterministic Simulation**:
- Each client runs independent 3D game loop
- Shared seed used for missile sequence via seedrandom
- Missile words selected from word-list dictionary using shared seed
- Each player has independent missile stream in their 3D scene
- Server does NOT track missile positions
- All players see their own ship at bottom center of their 3D view
- Opponent ships displayed as smaller 3D models in sidebar or minimap

**Player Elimination**:
- Player eliminated when ship health reaches 0
- Server broadcasts elimination to other players
- Game over when only one player remains
- Winner determined by last player standing
- Match results stored in database

**Score Synchronization**:
- Players send score_update every 2 seconds
- Server broadcasts to all other players in room
- missile_intercepted events broadcast for visual effects only

**Private Room Logic**:
- Room creator can configure game settings
- Invite code generated as 6-character alphanumeric string
- Password protection optional
- Room creator can kick players before game starts
- Room expires 1 hour after creation if game not started
- Room deleted after game completion

### 4.11 Tournament System Logic

**Tournament Structure**:
- Tournament types: Single Elimination, Double Elimination
- Bracket sizes: 8, 16, 32, 64 participants
- Tournament phases: Registration, Scheduled, In Progress, Completed
- Match rounds: Round 1, Round 2, Quarterfinals, Semifinals, Finals (adjusted based on bracket size)
- Double Elimination includes Winners Bracket and Losers Bracket
- Story Mode tournaments supported with story selection

**Tournament Creation**:
- Organizer creates tournament via POST /api/tournaments/create
- Required fields: name, description, gameMode, bracketType, maxParticipants, registrationDeadline, startDate
- Optional fields: entryFee, prizePool, customRules, storyId (for Story Mode tournaments)
- Tournament ID generated and stored in database
- Tournament status set to Registration
- Organizer assigned as tournament owner

**Tournament Registration**:
- Users register via POST /api/tournaments/{tournamentId}/register
- Registration allowed until registrationDeadline or maxParticipants reached
- User added to participants list in database
- Registration confirmation sent to user
- Users can unregister before tournament starts via POST /api/tournaments/{tournamentId}/unregister

**Bracket Generation**:
- Triggered when organizer starts tournament or at scheduled start time
- Participants randomly seeded into bracket positions
- Match pairings generated based on bracket type
- All matches created with status: scheduled
- Bracket structure stored in database
- Tournament status changed to In Progress

**Match Scheduling**:
- Round 1 matches scheduled immediately at tournament start
- Subsequent rounds scheduled after previous round completes
- Match start times calculated based on estimated match duration
- Participants notified of upcoming matches
- Match data stored in database with matchId, tournamentId, round, player1Id, player2Id, status, scheduledTime, storyId (if Story Mode)

**Match Execution**:
- Both players join Tournament Match Lobby at scheduled time
- Match uses standardized game settings from tournament configuration
- Shared seed generated: tournamentId + matchId + timestamp
- Both players play identical missile sequence for fairness
- Story Mode tournaments: both players experience same story with synchronized narration
- Match result determined by higher score or last player standing
- Match result submitted to server via POST /api/tournaments/matches/{matchId}/result
- Server validates result and updates match status to completed
- Winner advances to next round in bracket

**Match Result Validation**:
- Server receives match results from both clients
- Compare scores and session data for consistency
- Detect discrepancies and flag for review
- In case of disconnect, player who remained connected wins by default
- Forfeit recorded if player exits match early
- Match result stored in database with winner, loser, scores, timestamp

**Bracket Advancement**:
- Winner of match advances to next round
- Loser eliminated (Single Elimination) or moves to Losers Bracket (Double Elimination)
- Next round matches generated when current round completes
- Bracket visualization updated in real-time
- Participants notified of next match schedule

**Tournament Completion**:
- Tournament completes when Finals match finishes
- Final standings calculated: 1st, 2nd, 3rd places
- Prize distribution calculated based on prizePool settings
- Tournament status changed to Completed
- Results published on Tournament Details Screen
- Winners announced and prizes awarded
- Tournament data archived in database

**Tournament Leaderboard**:
- Separate leaderboard for tournament winners
- Display tournament name, winner, date, prize
- Filter by game mode and time range
- Click tournament to view full bracket and results

**Tournament Notifications**:
- Registration confirmation
- Tournament start reminder (24 hours, 1 hour before)
- Match schedule notification
- Match start reminder (15 minutes before)
- Match result notification
- Tournament completion and final standings
- All notifications stored in database and sent via in-app and email

**Tournament Data Persistence**:
- tournaments table: tournamentId, name, description, gameMode, bracketType, maxParticipants, registrationDeadline, startDate, status, organizerId, entryFee, prizePool, storyId, createdAt
- tournament_participants table: tournamentId, userId, registrationDate, status
- tournament_matches table: matchId, tournamentId, round, player1Id, player2Id, winnerId, player1Score, player2Score, status, scheduledTime, completedAt, storyId
- tournament_brackets table: tournamentId, bracketData (JSON structure)
- tournament_results table: tournamentId, rank, userId, prize, completedAt

**Tournament Rules Enforcement**:
- Standardized game settings for all matches
- Adaptive difficulty disabled in tournament matches
- Spaced repetition disabled for fairness
- Shared seed ensures identical missile sequences
- Story Mode tournaments use same story for all matches
- Anti-cheat measures: session validation, score verification
- Disconnect handling: remaining player wins by default
- Forfeit penalty: automatic loss and potential tournament ban

**Tournament Analytics**:
- Track tournament participation rates
- Analyze match completion times
- Monitor player performance across tournaments
- Generate tournament reports for organizers
- All analytics data stored in database

### 4.12 Leaderboard System Logic

**Leaderboard Structure**:
- Separate leaderboards for each game mode: Survival, Time Attack, Zen, Boss Mode, Hardcore, Speedrun, Story Mode
- Time range filters: All Time, This Month, This Week, Today
- **Each player appears only once per mode with their highest score (or fastest time for Speedrun)**
- Each leaderboard entry contains: leaderboardId, userId, username, mode, score, wpm, accuracy, missilesIntercepted, completionTime (Speedrun only), storyTitle (Story Mode only), timestamp, rank, previousRank

**Score Submission Flow**:
- **Critical: Score submission must execute synchronously and block until completion**
- On session end (game over or exit), client immediately submits score via POST /api/leaderboard/submit with payload: {mode, score, wpm, accuracy, missilesIntercepted, completionTime, storyId, storyCompletionPercentage, sessionId, timestamp}
- **Critical: Client must wait for server response before displaying Game Over Screen**
- Server validates JWT token and session data
- Server checks for duplicate submissions using sessionId
- **Critical: Server must use database transaction to ensure atomic operations**
- **Server checks if player already has entry for this mode**
- **If existing entry found, compare scores (or completion times for Speedrun)**
- **Only update leaderboard entry if new score is higher (or time is faster for Speedrun)**
- **If no existing entry, insert new leaderboard entry into database within transaction**
- **Critical: Server must verify successful database write before proceeding**
- Server triggers rank recalculation for affected mode and time ranges within same transaction
- **Critical: Server must commit transaction only after all operations succeed**
- **Critical: Server must rollback transaction if any operation fails**
- Server returns response: {success, newRank, previousRank, rankChange, isPersonalBest, scoreUpdated}
- **Critical: Client must handle response and display rank information on Game Over Screen**
- **Critical: Client must implement retry logic with exponential backoff if submission fails**
- **Critical: Client must display error message if all retry attempts fail**
- **Critical: All database write operations must include error handling and logging**
- **Critical: Server must validate all required fields present in submission payload**
- **Critical: Server must return detailed error messages for debugging**

**Rank Calculation Logic**:
- Ranks calculated dynamically based on score ordering (descending) or completion time ordering (ascending for Speedrun)
- **Rank = COUNT(distinct users with higher score/faster time) + 1**
- **Each userId counted only once per mode**
- Rank recalculation triggered on new score submission or score update
- Previous rank stored for rank change calculation
- Rank change = previousRank - newRank (positive = improvement, negative = decline)
- **Critical: Rank calculation query must execute within database transaction**
- **Critical: Rank calculation must handle edge cases (first entry, tied scores)**
- **Critical: Rank calculation must use proper WHERE clauses for mode and time range**
- **Critical: Rank calculation must complete within 2 seconds or timeout**

**Leaderboard Query Optimization**:
- Database indexes on: mode, timestamp, score, completionTime, userId
- Composite index on (mode, userId) for unique player per mode enforcement
- Composite index on (mode, timestamp, score) for efficient filtering and sorting
- **Critical: Verify all database indexes exist and are active**
- Leaderboard entries cached in memory for 60 seconds
- Cache invalidated on new score submission or update for affected mode
- Pagination implemented with LIMIT and OFFSET
- Top 100 entries loaded per page
- **Critical: Add query performance monitoring to identify slow queries**
- **Critical: Implement query result caching with proper invalidation strategy**
- **Critical: Add connection pooling to handle concurrent leaderboard requests**
- **Critical: Monitor database connection pool usage and adjust size if needed**

**Personal Best Tracking**:
- Server compares new score with user's previous best for same mode
- Personal best stored in user_stats table: userId, mode, bestScore, bestWpm, bestAccuracy, bestCompletionTime, achievedAt
- isPersonalBest flag returned in submission response if new score exceeds previous best
- Personal best updated in database on new record
- **Critical: Personal best comparison must handle null/undefined previous bests**
- **Critical: Personal best update must occur within same transaction as leaderboard entry**

**Rank Change Display**:
- Rank change indicator displayed on Game Over Screen
- Green up arrow with position change for improvement
- Red down arrow with position change for decline
- No indicator if rank unchanged
- Rank change animation plays on Game Over Screen

**Leaderboard Refresh Logic**:
- Auto-refresh every 60 seconds when viewing Leaderboard View
- Manual refresh button available
- Real-time updates via WebSocket for top 10 changes (optional)
- Refresh fetches latest leaderboard data from server
- Client-side cache updated on refresh
- **Critical: Refresh mechanism must properly fetch and update displayed data**
- **Critical: Add loading state during refresh to prevent UI flickering**
- **Critical: Implement error handling for failed refresh attempts**
- **Critical: Display error message with retry option if refresh fails**

**Time Range Filtering**:
- All Time: no timestamp filter
- This Month: timestamp >= start of current month
- This Week: timestamp >= start of current week (Monday)
- Today: timestamp >= start of current day (00:00:00)
- Time range filter applied in database query
- Separate rank calculation for each time range
- **Each player still appears only once per mode per time range with their best score in that period**
- **Critical: Verify timestamp filtering logic correctly calculates time boundaries**
- **Critical: Ensure timezone handling is consistent across client and server**

**Leaderboard Data Retention**:
- All Time leaderboard entries retained indefinitely
- Time-ranged entries archived after period expires
- Archived entries moved to separate table for historical analysis
- Active leaderboard table optimized for current period queries

**Leaderboard Display Fix**:
- **Critical: Add comprehensive error handling in leaderboard data fetching logic**
- **Critical: Implement fallback display when API returns empty or malformed data**
- **Critical: Add client-side validation to ensure score data structure matches expected format**
- **Critical: Verify API endpoint /api/leaderboard correctly queries database and returns formatted JSON**
- **Critical: Add detailed logging at each step of data flow: database query → API response → client rendering**
- **Critical: Implement retry mechanism with exponential backoff for failed API calls**
- **Critical: Add loading spinner or skeleton UI while fetching leaderboard data**
- **Critical: Verify database connection pool is properly configured and not exhausted**
- **Critical: Check for SQL injection vulnerabilities in leaderboard queries**
- **Critical: Ensure proper JOIN operations if username stored in separate users table**
- **Critical: Add unit tests for leaderboard data transformation logic**
- **Critical: Implement integration tests for end-to-end leaderboard flow**
- **Critical: Add monitoring and alerting for leaderboard API failures**
- **Critical: Verify CORS configuration allows leaderboard API requests**
- **Critical: Check authentication middleware does not block leaderboard view for public access**
- **Critical: Add database migration to ensure leaderboard table schema matches application expectations**
- **Critical: Verify all required columns exist in leaderboard table with correct data types**
- **Critical: Add data seeding script to populate test leaderboard data for development**
- **Critical: Implement graceful degradation if leaderboard service is temporarily unavailable**
- **Critical: Add client-side caching with TTL to reduce server load**
- **Critical: Verify API response includes proper HTTP status codes and error messages**
- **Critical: Ensure score submission API endpoint validates all required fields**
- **Critical: Add server-side logging for all score submission attempts**
- **Critical: Implement database transaction rollback on any submission failure**
- **Critical: Add automated tests for score submission and leaderboard update flow**
- **Critical: Monitor score submission success rate and alert on anomalies**
- **Critical: Verify leaderboard entry includes sessionId to prevent duplicates**
- **Critical: Add timestamp validation to prevent backdated score submissions**
- **Critical: Implement rate limiting on score submission endpoint**
- **Critical: Add database constraints to ensure data integrity**
- **Critical: Verify foreign key relationships between users and leaderboard tables**
- **Critical: Add database backup and recovery procedures**
- **Critical: Implement database replication for high availability**
- **Critical: Add performance benchmarks for leaderboard queries**
- **Critical: Monitor database query execution time and optimize slow queries**
- **Critical: Implement database query caching for frequently accessed data**
- **Critical: Add database connection retry logic with exponential backoff**
- **Critical: Verify database connection timeout settings are appropriate**
- **Critical: Implement database health checks and monitoring**
- **Critical: Add alerting for database connection failures**
- **Critical: Verify leaderboard API endpoint returns consistent data format**
- **Critical: Add API versioning to prevent breaking changes**
- **Critical: Implement API documentation for leaderboard endpoints**
- **Critical: Add API rate limiting to prevent abuse**
- **Critical: Verify API authentication and authorization logic**
- **Critical: Implement API request validation and sanitization**
- **Critical: Add API error handling and logging**
- **Critical: Monitor API response times and optimize slow endpoints**
- **Critical: Implement API caching for frequently accessed data**
- **Critical: Add API health checks and monitoring**
- **Critical: Verify API CORS configuration allows cross-origin requests**
- **Critical: Implement API security best practices**
- **Critical: Add unique constraint on (userId, mode) in leaderboard table to enforce one entry per player per mode**
- **Critical: Implement UPSERT logic (INSERT ON CONFLICT UPDATE) for score submissions**
- **Critical: Verify leaderboard queries use DISTINCT userId or GROUP BY userId**
- **Critical: Add database trigger or application logic to prevent duplicate userId entries per mode**

### 4.13 Analytics and Telemetry

**Event Types Tracked**:
- session_start, missile_intercepted, missile_missed, missile_hit_ship, first_intercept, milestone, boss_encounter, speedrun_complete, story_started, story_completed, story_progress, session_end, rage_quit, game_exited, friend_added, replay_saved, score_shared, word_bank_created, learning_goal_set, vocabulary_milestone, leaderboard_rank_improved, personal_best_achieved, tournament_registered, tournament_match_started, tournament_match_completed, tournament_won

**Event Processing**:
- In-memory queue with async flush every 5 seconds
- Batch POST to /api/events
- Immediate flush on session_end, rage_quit, or game_exited
- rage_quit and game_exited use navigator.sendBeacon on beforeunload
- Events persisted to database for historical analysis
- Retry logic with exponential backoff on API failure (max 3 retries)
- Analytics events stored in database

### 4.14 Friend System Logic

**Friend Request Flow**:
- User searches for friend by username
- Send friend request via POST /api/friends/request
- Target user receives notification
- Target user accepts or declines via POST /api/friends/respond
- On acceptance, both users added to each other's friend list
- Friend list stored server-side, synced to client on login
- Friend notifications stored in database

**Friend Activity Feed**:
- Server tracks friend session_end events
- Feed displays last 20 friend activities
- Activity types: new high score, achievement unlocked, game completed, leaderboard rank improved, tournament participation, tournament victory, story completed
- Real-time updates via WebSocket when friend online

**Multiplayer Challenge**:
- User clicks Challenge Friend button
- System creates private room with 2-player capacity
- Invite sent to friend via notification
- Friend accepts to join room
- Game starts when both players ready

### 4.15 Replay System Logic

**Recording**:
- Replay recording starts on game start
- Frame data captured every 16ms:
  - Timestamp
  - Player ship 3D position and rotation
  - Active missiles (text, 3D position, typed prefix)
  - Active interceptors (3D position, targetId)
  - Enemy ships (3D position, rotation, lane)
  - Explosions (3D position, frame)
  - Score, combo, health, WPM
  - Input events (key, timestamp)
  - Camera position and rotation
  - Narration audio timestamp (Story Mode)
  - Narrative text position (Story Mode)
- Recording stops on game over or exit
- Replay data compressed and saved to localStorage
- Optional upload to server for sharing
- Replay data stored in database with compression

**Playback**:
- Load replay file from localStorage or server
- Reconstruct 3D game state frame-by-frame
- Render on 3D scene using same rendering logic
- Playback controls allow speed adjustment and seeking
- Timeline scrubber shows progress through replay
- Camera follows recorded camera movements or allows free camera mode
- Story Mode replays include narration audio playback
- Narrative text displayed during Story Mode replay

**Sharing**:
- Generate unique replay ID on server upload
- Create shareable URL: /replay/{replayId}
- Replay accessible to anyone with link
- Replay metadata displayed before playback
- Option to download replay file

### 4.16 Custom Word Bank Logic

**Word Bank Structure**:
- Word bank contains: id, name, description, category, words array, creatorId, isPublic, createdAt
- All word bank data persisted to database

**Word Bank Selection**:
- User selects word bank in Focus Session mode
- Selected word bank overrides word-list dictionary
- Missile spawning uses selected word bank exclusively
- Spaced repetition applies to words in selected bank
- User preferences stored in database

**Community Sharing**:
- User marks word bank as public
- Public word banks appear in community browser
- Other users can clone public word banks
- Cloned word banks become independent copies
- Original creator credited in cloned bank metadata
- Clone relationships tracked in database

**Import/Export**:
- CSV format: word,difficulty (one per line)
- JSON format: array of {text, difficulty} objects
- Text format: one word per line (default difficulty 3)
- Export preserves all metadata and word data

### 4.17 Learning Path Recommendations

**Recommendation Engine**:
- Analyze last 30 days of session data from database
- Identify weak areas
- Generate recommendations
- Recommendations persisted to database for tracking

**Goal Tracking**:
- User sets goals: target WPM, target accuracy, daily practice time
- System tracks progress toward goals
- Display progress percentage and estimated completion date
- Send notifications when goals achieved
- Suggest new goals based on current performance
- All goal data stored in database

**Practice Plan Generation**:
- Weekly plan based on goals and weak areas
- Recommend specific game modes and categories per day
- Suggest session duration and frequency
- Adjust plan based on actual performance
- Practice plans stored in database

### 4.18 Progress Reports Analytics

**Data Aggregation**:
- Aggregate session data by time range from database
- Calculate statistics

**Trend Analysis**:
- Calculate linear regression for WPM and accuracy trends
- Identify improvement rate
- Detect performance plateaus
- Highlight significant changes
- Track vocabulary expansion over time

**Comparative Analysis**:
- Compare performance across game modes
- Compare performance across categories
- Compare current period to previous period
- Show percentile ranking among all users

**Export Functionality**:
- Generate PDF report with charts and statistics
- Export raw data as CSV for external analysis
- Create shareable report link with public access
- Exported reports stored in database

### 4.19 Offline Mode Support

**Local Data Sync**:
- Game playable without internet connection
- Word stats and progress stored in localStorage
- Queue analytics events locally when offline
- Sync queued events to server when connection restored
- Conflict resolution: server data takes precedence on sync
- Sync status tracked in database

**Offline Functionality**:
- All single-player game modes available offline
- word-list dictionary cached locally for offline access
- Custom word banks accessible offline
- Story Mode stories cached locally for offline play
- Leaderboard and multiplayer features disabled offline
- Tournament features disabled offline
- Offline indicator displayed in UI
- Automatic reconnection attempt on network restore

### 4.20 API Retry Logic

**Retry Mechanism**:
- Exponential backoff strategy for failed API calls
- Initial retry delay: 1 second
- Maximum retry attempts: 3
- Backoff multiplier: 2x per retry
- Retry on network errors and 5xx server errors
- No retry on 4xx client errors (except 429 rate limit)

**Error Handling**:
- Display user-friendly error messages
- Log detailed error information for debugging
- Graceful degradation when API unavailable
- Queue critical operations for retry
- Notify user of persistent connection issues
- Error logs stored in database

### 4.21 Database Schema Overview

**Core Tables**:
- users, sessions, word_stats, leaderboards, user_stats, friendships, friend_activities, rooms, room_participants, matches, replays, word_banks, word_bank_entries, learning_goals, recommendations, practice_plans, analytics_events, notifications, user_preferences, share_events, exported_reports, sync_status, error_logs, vocabulary_stats, stories, story_completions

**Story Tables**:
- stories: storyId, title, duration, difficulty, genre, synopsis, narrativeText, narratorVoice, wordCount, timingData, createdAt
- story_completions: userId, storyId, completionPercentage, score, wpm, accuracy, completedAt

**Tournament Tables**:
- tournaments: tournamentId, name, description, gameMode, bracketType, maxParticipants, registrationDeadline, startDate, status, organizerId, entryFee, prizePool, customRules, storyId, createdAt
- tournament_participants: tournamentId, userId, registrationDate, status
- tournament_matches: matchId, tournamentId, round, player1Id, player2Id, winnerId, player1Score, player2Score, status, scheduledTime, completedAt, sharedSeed, storyId
- tournament_brackets: tournamentId, bracketData (JSON)
- tournament_results: tournamentId, rank, userId, prize, completedAt
- tournament_notifications: notificationId, tournamentId, userId, type, message, sentAt, readAt

**Leaderboard Tables**:
- leaderboards: leaderboardId, userId, username, mode, score, wpm, accuracy, missilesIntercepted, completionTime, storyTitle, storyCompletionPercentage, timestamp, rank, previousRank, sessionId
- user_stats: userId, mode, bestScore, bestWpm, bestAccuracy, bestCompletionTime, achievedAt
- Indexes: (mode, timestamp, score), (mode, completionTime), (userId, mode), (sessionId)
- **Unique constraint on (userId, mode) to enforce one entry per player per mode**
- **Critical: Verify all indexes exist and are optimized**
- **Critical: Add unique constraint on sessionId to prevent duplicate submissions**
- **Critical: Add foreign key constraints for data integrity**
- **Critical: Implement database triggers for automatic rank calculation**
- **Critical: Add database views for common leaderboard queries**

**Data Persistence Rules**:
- All user-generated content persisted to database
- Session data saved on game over or exit
- Word stats updated after each session
- **Critical: Leaderboard data must be stored immediately and atomically on session end or exit**
- **Critical: Leaderboard entries must include all required fields with proper data types**
- **Critical: Database writes must use transactions to ensure atomicity**
- **Critical: Failed database writes must trigger rollback and retry**
- **Critical: Each player can have only one leaderboard entry per mode, updated with highest score**
- Friend relationships persisted to database
- Friend activity feed populated from database
- Private room data stored in database
- Match results saved to database on completion
- Tournament data persisted throughout lifecycle
- Tournament match results validated and stored
- Replay data compressed before storage
- Word bank data persisted to database
- Learning goals tracked in database
- Recommendations stored in database
- Practice plans saved to database
- User preferences persisted to database
- Story data and completions stored in database
- Database backups performed daily
- Data retention: user data indefinite, analytics events 2 years, error logs 90 days

## 5. Exception and Edge Cases

| Scenario | Handling |
|----------|----------|
| Tab hidden during gameplay | Pause rAF and 3D rendering, pause narration audio (Story Mode), reset lastTime to null on visibilitychange |
| DeltaTime spike on tab restore | Cap deltaTime at 0.1 seconds (100ms) |
| All lanes occupied | Skip spawn, do not create enemy ship or missile |
| Rapid typing (double-registration) | keydown handler must be idempotent per event |
| Font not loaded | Wait for document.fonts.load with 2-second timeout, fallback to Courier New |
| Server unreachable (analytics) | Queue events locally, retry with exponential backoff |
| WebSocket disconnect (multiplayer) | Display connection lost message, allow reconnection attempt |
| localStorage quota exceeded | Gracefully handle error, continue without persistence |
| Invalid JWT token | Clear token, redirect to Authentication Screen |
| Word bank empty for difficulty range | Fallback to word-list dictionary without filtering |
| Boss missile overlaps with regular missile | Clear overlapping lanes before spawning boss missile |
| Speedrun sequence generation fails | Fallback to default fixed sequence from word-list |
| Hardcore mode immediate death | Display game over screen with 0 score, save score to leaderboard |
| Friend request to non-existent user | Display error message, do not send request |
| Duplicate friend request | Display message indicating request already sent |
| Private room invite code invalid | Display error message, do not join room |
| Private room full | Display message indicating room at capacity |
| Replay file corrupted | Display error message, cannot load replay |
| Replay upload fails | Save locally, retry with exponential backoff |
| Custom word bank import invalid format | Display error message with format requirements |
| Custom word bank exceeds size limit | Display message, limit to 10000 words |
| Social sharing API unavailable | Display error message, allow copy link fallback |
| Learning path data insufficient | Display message indicating more sessions needed |
| Progress report generation timeout | Display partial report with available data |
| Export PDF generation fails | Fallback to CSV export option |
| Network connection lost during gameplay | Switch to offline mode, queue sync operations, save session data locally |
| Database write failure | Retry with exponential backoff, log error |
| API rate limit exceeded | Display message, implement client-side throttling |
| CDN asset load failure | Fallback to origin server, retry failed assets |
| Database connection pool exhausted | Queue requests, display loading indicator |
| Database query timeout | Retry query, fallback to cached data if available |
| Concurrent session data updates | Use optimistic locking, last write wins |
| Word stats sync conflict | Server data takes precedence, use higher attempts count as tiebreaker |
| Replay data exceeds storage limit | Compress further, truncate oldest frames if necessary |
| User account deletion | Cascade delete all user data, anonymize leaderboard entries |
| Interceptor missile target destroyed by another interceptor | Remove orphaned interceptor, no score penalty |
| Multiple missiles with same word active | Lock to first spawned missile on partial match |
| Boss missile partially typed when boss defeated | Clear progress, do not award partial score |
| Player ship destroyed during boss encounter | Boss retreats, no bonus score awarded, save session data |
| Missile spawns outside scene bounds | Clamp spawn position to valid coordinates |
| Explosion animation overlaps with new missile | Render explosions on separate layer below missiles |
| Health bar rendering with fractional health | Round health to nearest integer for display |
| Combo multiplier exceeds maximum | Cap at 4.0, do not allow overflow |
| WPM calculation with zero time elapsed | Default to 0 WPM, avoid division by zero |
| Accuracy calculation with zero keystrokes | Default to 100% accuracy |
| Star particle system out of sync | Reset star positions on game start |
| Enemy ship drifts outside lane boundaries | Clamp ship position to lane min/max |
| Interceptor missile speed too slow | Ensure minimum speed of 12 units/sec |
| Critical threat indicator on multiple missiles | Apply to all missiles meeting threshold |
| Camera shake during replay playback | Disable camera shake in replay mode |
| Ship damage visual persists after game over | Clear damage state on game restart |
| Enemy ship collision with player ship | No collision detection between ships, missiles only |
| Multiple enemy ships in same lane | Allow multiple ships per lane, maintain spacing |
| Exit button clicked during gameplay | Pause game and 3D rendering, stop narration (Story Mode), save session data immediately, submit score to leaderboard, display confirmation dialog, handle user choice |
| Exit confirmation dialog dismissed | Resume game, resume narration (Story Mode), continue gameplay |
| Content exceeds viewport height | Enable scrolling for affected screen, maintain smooth scroll behavior |
| Scrollbar not visible on some browsers | Apply custom scrollbar styling with fallback to native scrollbar |
| User exits game via exit button | Track game_exited event with exit method, save session data, submit score to leaderboard |
| word-list package fails to load | Display error message, fallback to minimal hardcoded word list |
| word-list dictionary empty after filtering | Fallback to full dictionary without filters |
| Word categorization fails | Assign word to general category |
| Duplicate words in word-list | Deduplicate during initialization |
| Word label exceeds maximum display width | Truncate word display, preserve full word for typing |
| Vocabulary stats calculation error | Log error, display last known stats |
| 3D model fails to load | Display error message, use placeholder geometry (box or sphere) |
| Texture fails to load | Use fallback solid color material |
| WebGL context lost | Display error message, attempt context restoration, reload page if necessary |
| Frame rate drops below 30 FPS | Reduce particle counts, disable non-essential visual effects |
| Memory leak from undisposed 3D objects | Implement proper disposal in cleanup functions |
| Camera position invalid | Reset camera to default position (0, 5, 15) |
| Lighting intensity too high | Cap light intensity at maximum safe values |
| Particle system exceeds maximum count | Limit particle emission, recycle oldest particles |
| 3D collision detection false positive | Increase collision threshold slightly, add cooldown period |
| Missile 3D position NaN or Infinity | Reset missile to valid spawn position or remove from scene |
| Enemy ship 3D rotation invalid | Reset rotation to default values |
| Explosion effect not visible | Increase particle size and emission rate |
| Word label sprite not facing camera | Ensure billboard behavior enabled for all text sprites |
| 3D scene rendering black screen | Check lighting setup, ensure at least one light source active |
| Shadow artifacts on 3D models | Adjust shadow bias and near/far planes |
| Transparent materials rendering incorrectly | Ensure correct render order, enable depth write for opaque objects |
| Post-processing effects causing performance issues | Make post-processing optional, disable on low-end devices |
| **Leaderboard submission fails** | **Retry with exponential backoff (max 3 attempts), display error message if all retries fail, log detailed error for debugging** |
| **Duplicate leaderboard submission** | **Server rejects duplicate using sessionId, return existing rank without creating new entry** |
| **Player submits multiple scores for same mode** | **Server compares with existing entry, only updates if new score is higher (or time is faster for Speedrun)** |
| **Leaderboard rank calculation timeout** | **Use cached ranks, recalculate asynchronously, return estimated rank with indicator** |
| **Leaderboard cache invalidation fails** | **Force cache refresh on next query, log error for monitoring** |
| **Personal best comparison fails** | **Log error, continue without personal best indicator, retry on next submission** |
| **Rank change calculation error** | **Default to no rank change, log error, display rank without change indicator** |
| **Leaderboard pagination out of bounds** | **Return empty results, display message indicating end of list** |
| **Time range filter invalid** | **Default to All Time filter, log warning** |
| **Leaderboard mode filter invalid** | **Default to Survival mode, log warning** |
| **Leaderboard auto-refresh fails** | **Display error message, allow manual refresh, retry on next interval** |
| **WebSocket connection for real-time updates fails** | **Fallback to polling every 60 seconds, display connection status** |
| **Leaderboard data corruption** | **Rebuild leaderboard from session data, notify administrators** |
| **User stats table out of sync** | **Recalculate personal bests from leaderboard entries, update user_stats table** |
| **Archived leaderboard entries inaccessible** | **Display message, provide link to historical data if available** |
| **Leaderboard API returns empty array** | **Display message: No scores available yet, be the first to play!** |
| **Leaderboard API returns malformed JSON** | **Log error, display fallback message, retry request with exponential backoff** |
| **Leaderboard component receives null/undefined data** | **Render empty state with call-to-action to play game** |
| **Leaderboard table rendering fails** | **Catch rendering error, display error boundary with retry button** |
| **Score data missing required fields** | **Validate data structure, log missing fields, skip invalid entries, display error message** |
| **Username missing in leaderboard entry** | **Display Anonymous or Player {userId} as fallback** |
| **Leaderboard database query fails** | **Return cached data if available, otherwise return empty array with error flag, log error** |
| **Leaderboard JOIN operation fails** | **Ensure users table exists and userId foreign key is valid, log error, retry query** |
| **Leaderboard index missing or corrupted** | **Rebuild index, log warning, continue with slower query** |
| **Concurrent leaderboard writes cause deadlock** | **Implement row-level locking, retry transaction with exponential backoff** |
| **Leaderboard cache out of sync with database** | **Implement cache invalidation on write, add cache versioning, force refresh** |
| **Client-side leaderboard state not updating** | **Force re-render on data change, verify React state management** |
| **Leaderboard scroll position resets on refresh** | **Preserve scroll position in component state** |
| **Leaderboard pagination state lost** | **Store pagination state in URL query parameters** |
| **Score submission database transaction fails** | **Rollback transaction, retry with exponential backoff, log detailed error** |
| **Score submission validation fails** | **Return validation error to client, do not create leaderboard entry** |
| **Score submission rate limit exceeded** | **Return rate limit error, implement client-side throttling** |
| **Score submission with invalid JWT token** | **Return authentication error, redirect to login** |
| **Score submission with expired session** | **Return session expired error, prompt re-authentication** |
| **Score submission with tampered data** | **Reject submission, log security event, potentially ban user** |
| **Database connection lost during score submission** | **Retry connection with exponential backoff, queue submission for retry** |
| **Database transaction timeout during score submission** | **Rollback transaction, retry with shorter timeout, log error** |
| **Rank recalculation takes too long** | **Use asynchronous rank calculation, return estimated rank immediately** |
| **Personal best update fails** | **Log error, continue with leaderboard entry creation, retry personal best update** |
| **Leaderboard entry creation succeeds but rank calculation fails** | **Log error, return success with estimated rank, trigger background rank recalculation** |
| **Unique constraint violation on (userId, mode)** | **Trigger UPSERT logic, update existing entry if new score is higher** |
| Tournament registration after deadline | Display error message, registration closed |
| Tournament registration when full | Display message indicating tournament at capacity |
| Tournament start with insufficient participants | Cancel tournament, notify participants, refund entry fees |
| Tournament bracket generation fails | Retry generation, notify organizer of error |
| Tournament match scheduling conflict | Reschedule conflicting matches, notify participants |
| Tournament match no-show | Award win to present player after 5-minute grace period |
| Tournament match disconnect during play | Remaining player wins by default, match result recorded |
| Tournament match result discrepancy | Flag for manual review, notify organizer |
| Tournament match forfeit | Award win to opponent, record forfeit in match data |
| Tournament bracket advancement error | Recalculate bracket, notify affected participants |
| Tournament completion with tied scores | Use tiebreaker rules (accuracy, completion time) |
| Tournament prize distribution failure | Retry distribution, notify organizer and winners |
| Tournament data corruption | Restore from backup, notify organizer |
| Tournament notification delivery failure | Retry with exponential backoff, log failed notifications |
| Tournament organizer account deleted | Transfer ownership to co-organizer or cancel tournament |
| Tournament shared seed generation fails | Fallback to timestamp-based seed |
| Tournament match lobby timeout | Auto-start match or forfeit no-show player |
| Tournament bracket view rendering error | Display error message, provide text-based bracket |
| Tournament leaderboard query timeout | Use cached data, retry query asynchronously |
| Tournament analytics calculation error | Log error, display partial analytics |
| Story not found in database | Display error message, return to Story Selection Screen |
| Story narrative text empty or corrupted | Display error message, cannot start Story Mode |
| AI narration audio fails to load | Display error message, fallback to text-only mode |
| AI narration audio playback error | Pause game, display error message, allow retry |
| Story timing data missing or invalid | Use default timing based on word count |
| Story completion percentage calculation error | Default to 0%, log error |
| Story Mode session interrupted | Save partial progress, submit score to leaderboard, allow resume from last checkpoint |
| Story Mode leaderboard submission with incomplete story | Accept submission, display completion percentage |
| Story Mode tournament with missing story | Cancel tournament, notify participants |
| Story Mode replay without narration audio | Display text-only replay, indicate audio unavailable |
| Narrative text display overflow | Truncate text, add scrolling to narrative panel |
| Story progress bar rendering error | Hide progress bar, log error |
| Multiple stories with same title | Differentiate by storyId, display unique identifier |
| Story filter returns no results | Display message indicating no stories match criteria |
| Story selection screen load timeout | Display error message, retry loading stories |
| Story cache invalidation fails | Force reload stories from database |
| Story completion data sync conflict | Server data takes precedence, merge completion records |
| Story Mode offline play without cached story | Display error message, require online connection |

## 6. Acceptance Criteria

1. 3D scene renders at stable 60 FPS on modern browsers with WebGL support
2. Three.js scene initializes correctly with camera, lighting, and fog
3. Player spaceship 3D model loads and renders at bottom center of viewport
4. Enemy spaceship 3D models spawn at far distance and move toward camera
5. Missiles render as 3D projectiles with glowing trails
6. Interceptor missiles launch from player ship and home toward targets in 3D space
7. Word labels render as 3D text sprites above missiles with billboard behavior
8. Collision detection works correctly in 3D space using bounding spheres
9. Explosion effects render as 3D particle systems at collision points
10. Ship damage effects display 3D sparks and material changes
11. Star field particle system renders with parallax depth effect
12. Nebula clouds render as volumetric sprites in background
13. Distant planets rotate in far background
14. Dynamic lighting updates as ships move through 3D space
15. Point lights attached to ships illuminate surrounding area
16. PBR materials render realistically with metalness and roughness
17. Emissive materials glow correctly for engines and weapon ports
18. Transparent materials render correctly for cockpit glass
19. Camera maintains correct position and FOV throughout gameplay
20. Camera shake effect works in 3D space on missile hit (disabled in Zen and Story Mode)
21. 3D scene responds to window resize events
22. DPI scaling applied correctly for high-resolution displays
23. 3D models load from GLTF files without errors
24. Textures load and apply correctly to 3D models
25. Loading screen displays during 3D asset loading
26. Fallback placeholder models used if 3D model loading fails
27. Object pooling reduces garbage collection for missiles and particles
28. Frustum culling skips rendering off-screen objects
29. Geometry and material disposal prevents memory leaks
30. Frame rate monitoring adjusts particle counts dynamically
31. WebGL context loss handled gracefully with restoration attempt
32. Input handling responds within 16ms of keydown event
33. Combo system increments and resets according to specified rules
34. Ship health decreases on missile hit, game over triggers at 0 health
35. Adaptive difficulty adjusts spawn parameters based on player metrics
36. Word stats persist to database after each session
37. Spaced repetition surfaces due words with 3x weight
38. Danger mode activates at 6 active missiles, red alert overlay displays
39. Critical threat indicator reduces missile speed by 20% when conditions met
40. All seven game modes function correctly in 3D environment
41. Focus Session filters words by selected category or custom word bank
42. Authentication flow completes successfully, JWT stored
43. Word stats sync between client and server on login
44. **Leaderboard displays top 100 entries for selected mode and time range with each player appearing only once**
45. Leaderboard highlights current player row with bright cyan background
46. Leaderboard shows rank change indicators (up/down arrows)
47. Leaderboard mode selector tabs switch between game modes including Story Mode
48. Leaderboard time range filter updates rankings correctly
49. Leaderboard pagination controls navigate through all entries
50. Leaderboard auto-refreshes every 60 seconds
51. Leaderboard manual refresh button updates data immediately
52. **Score submission executes synchronously and blocks until server confirmation received**
53. **Score submission completes successfully before Game Over Screen displays**
54. **Score submission uses database transaction to ensure atomicity**
55. **Score submission validates all required fields before database write**
56. **Score submission returns new rank and rank change to client**
57. **Score submission implements retry logic with exponential backoff on failure**
58. **Score submission displays error message if all retry attempts fail**
59. **Score submission logs detailed error information for debugging**
60. **Score submission prevents duplicate entries using sessionId**
61. **Score submission updates existing leaderboard entry if new score is higher for same player and mode**
62. **Leaderboard enforces unique constraint on (userId, mode) to prevent duplicate player entries**
63. **Leaderboard queries use DISTINCT userId or GROUP BY userId to ensure one entry per player**
64. **Leaderboard rank calculation counts distinct users only**
65. Personal best indicator displays on Game Over Screen when achieved
66. Rank change animation plays on Game Over Screen
67. View Full Leaderboard button navigates to Leaderboard View
68. Leaderboard data persisted to database with correct indexes
69. Leaderboard rank calculation accurate for all modes
70. Speedrun leaderboard ranks by completion time ascending with one entry per player
71. Story Mode leaderboard displays story title and completion percentage with one entry per player
72. Leaderboard ties resolved by timestamp
73. Leaderboard cache invalidated on new score submission or update
74. Leaderboard query performance optimized with indexes
75. Personal best tracked and updated in user_stats table
76. Leaderboard entries archived after time period expires
77. Top 3 ranks display with gold, silver, bronze highlights
78. Leaderboard accessible from Mode Selection Screen
79. Leaderboard accessible from Game Over Screen
80. Leaderboard public access without authentication
81. Click username on leaderboard to view player profile
82. **Leaderboard displays player scores correctly with all required data fields visible**
83. **Leaderboard API endpoint returns properly formatted JSON with username, score, wpm, accuracy, and rank**
84. **Leaderboard component renders without errors when receiving valid data**
85. **Leaderboard shows loading state while fetching data from server**
86. **Leaderboard displays appropriate error message when data fetch fails**
87. **Leaderboard handles empty state gracefully with call-to-action message**
88. **Leaderboard score submission writes all required fields to database successfully**
89. **Leaderboard rank calculation executes correctly and updates in real-time**
90. **Leaderboard database queries complete within acceptable time limits**
91. **Leaderboard cache invalidation works correctly on new score submission**
92. **Leaderboard database transaction commits only after all operations succeed**
93. **Leaderboard database transaction rolls back if any operation fails**
94. **Leaderboard API includes comprehensive error handling and logging**
95. **Leaderboard client implements retry mechanism for failed API calls**
96. **Leaderboard displays loading spinner during data fetch**
97. **Leaderboard validates data structure before rendering**
98. **Leaderboard handles malformed API responses gracefully**
99. **Leaderboard database indexes verified and optimized**
100. **Leaderboard foreign key constraints ensure data integrity**
101. **Leaderboard unique constraint on sessionId prevents duplicates**
102. **Leaderboard automated tests verify end-to-end submission flow**
103. **Leaderboard monitoring alerts on submission failures**
104. **Leaderboard database backup and recovery procedures functional**
105. **Leaderboard unique constraint on (userId, mode) enforced in database**
106. **Leaderboard UPSERT logic implemented for score submissions**
107. **Leaderboard prevents multiple entries per player per mode**
108. Multiplayer rooms match players, shared seed generates deterministic missile streams
109. Analytics events flush every 5 seconds, rage_quit fires on beforeunload
110. Game over screen displays all required metrics and navigation options
111. Caps Lock does not break word matching
112. Backspace does not modify buffer
113. Tab restore does not cause missile teleporting in 3D space
114. Boss Mode spawns boss ship 3D model every 20 intercepted missiles
115. Armored missile 3D model requires 3 successful typings to destroy
116. Boss defeat awards bonus score and does not reset combo
117. Boss missile hit deducts 2 health points and resets combo
118. Hardcore mode starts with 1 health point and ends on first hit
119. Speedrun mode uses fixed sequence and tracks completion time
120. Speedrun leaderboard ranks by completion time with one entry per player
121. Speedrun missed missiles respawn without health penalty
122. Mode multipliers apply correctly to final score including Story Mode (1.3)
123. Enemy ships spawn at far distance in assigned lanes
124. Enemy ships move forward toward camera in 3D space
125. Enemy ships drift horizontally within lane boundaries
126. Missiles spawn from enemy ship 3D positions
127. Interceptor missiles launch from player ship on correct word match
128. Interceptor missiles travel toward specific target in 3D space
129. Collision detection works correctly between interceptor and enemy missile in 3D
130. Explosion animation plays on missile intercept with 3D particle effects
131. Ship damage visual feedback displays on hit with 3D effects
132. Star field particles move at varying speeds for parallax depth
133. 3D space-themed backgrounds render correctly on all screens
134. HUD overlay displays all required information with high-contrast colors
135. Health bar segments display correctly in HUD
136. Friend search returns matching users
137. Friend request sent and received successfully
138. Friend list displays with online status indicators
139. Friend activity feed updates in real-time
140. Friend activity feed displays leaderboard rank improvements
141. Friend activity feed displays tournament participation
142. Friend activity feed displays story completions
143. Challenge friend creates private room and sends invite
144. Private room creation generates unique invite code
145. Private room join via invite code works correctly
146. Private room settings apply to game session
147. Room creator can kick players before game start
148. Replay recording captures all 3D game state data including narration (Story Mode)
149. Replay playback reconstructs 3D game accurately
150. Replay speed adjustment works correctly
151. Replay timeline scrubber allows seeking
152. Replay sharing generates unique URL
153. Social sharing posts to selected platforms
154. Share image generated with correct statistics and 3D ship render
155. Custom word bank creation saves successfully to database
156. Custom word bank import supports CSV, JSON, text formats
157. Custom word bank export preserves all data
158. Community word banks browsable and cloneable
159. Selected word bank used in Focus Session
160. Learning path recommendations generated based on performance
161. Learning goals set and tracked correctly in database
162. Practice plan generated weekly
163. Progress reports display all statistics correctly with high-contrast charts
164. Progress report charts render accurately with bright colors
165. Progress report export to PDF works
166. Progress report export to CSV works
167. Trend analysis calculates improvement rate
168. Comparative analysis shows mode and category differences
169. All analytics events tracked and persisted to database
170. Analytics events retry with exponential backoff on failure
171. Offline mode allows gameplay without internet connection
172. Local data syncs to server when connection restored
173. Offline indicator displays when network unavailable
174. API calls retry automatically on network errors
175. Failed API calls display user-friendly error messages
176. Database persistence stores all user data
177. Historical analytics data queryable for reports
178. User account data stored in database on registration
179. Session data persisted to database on game over or exit
180. Word stats updated in database after each session
181. **Leaderboard data stored immediately and atomically on session end or exit**
182. **Leaderboard entries include all required fields with proper data types**
183. **Database writes use transactions to ensure atomicity**
184. **Failed database writes trigger rollback and retry**
185. **Each player has only one leaderboard entry per mode, updated with highest score**
186. Friend relationships persisted to database
187. Friend activity feed populated from database
188. Private room data stored in database
189. Match results saved to database on completion
190. Replay data compressed and stored in database
191. Word bank data persisted to database
192. Learning goals tracked in database
193. Recommendations stored in database
194. Practice plans saved to database
195. User preferences persisted to database
196. All database operations logged for audit trail
197. All text elements have sufficient contrast against backgrounds
198. Word labels on missiles clearly visible with dark background padding
199. All UI elements use high-contrast color schemes as specified
200. Stars in 3D particle system clearly visible with bright white color
201. Player and enemy ships have bright colored materials for clear visibility
202. All buttons have high-contrast backgrounds and text colors
203. Form inputs have clear borders and high-contrast text
204. HUD elements have semi-transparent dark backgrounds for readability
205. All status indicators use bright, easily distinguishable colors
206. Chart elements in analytics use bright colors on dark backgrounds
207. Multiple enemy ships can be visible simultaneously in 3D scene
208. Enemy ships pursue player ship by moving forward through 3D space
209. Each enemy ship launches missiles independently
210. Interceptor missiles target specific enemy missiles based on typed word
211. Exit button displays in top right corner of game HUD with high-contrast colors
212. Exit button clickable area minimum 40px x 40px
213. Exit button click pauses game and 3D rendering, stops narration (Story Mode), saves session data, submits score to leaderboard, displays confirmation dialog
214. Confirmation dialog displays with Yes and No options in high-contrast colors
215. Clicking Yes in confirmation dialog stops game and navigates to appropriate screen
216. Clicking No in confirmation dialog resumes game, resumes narration (Story Mode), closes dialog
217. Exit button hover effect displays brighter red background
218. game_exited analytics event tracked when user exits via button
219. All screens except 3D Game Scene are scrollable when content exceeds viewport height
220. Authentication Screen scrollable when form content exceeds viewport
221. Mode Selection Screen scrollable when mode cards exceed viewport
222. Story Selection Screen scrollable when story cards exceed viewport
223. Tournament Hub Screen scrollable when tournament list exceeds viewport
224. Tournament Creation Screen scrollable when configuration form exceeds viewport
225. Tournament Details Screen scrollable when content exceeds viewport
226. Tournament Match Lobby chat messages scrollable when exceeding viewport
227. Game Over Screen results overlay scrollable when content exceeds viewport
228. Leaderboard View table scrollable when entries exceed viewport
229. Friends Management Screen scrollable when friend list exceeds viewport
230. Private Room Creation Screen scrollable when settings exceed viewport
231. Replay Viewer Screen information panel scrollable when content exceeds viewport
232. Social Sharing Screen scrollable when sharing options exceed viewport
233. Custom Word Bank Manager scrollable when word banks exceed viewport
234. Learning Path Dashboard scrollable when recommendations exceed viewport
235. Progress Reports Screen scrollable when charts exceed viewport
236. Scrollbar styling consistent with space theme across all screens
237. Smooth scrolling enabled for better user experience
238. Mobile devices use native scrolling behavior
239. 3D Game Scene maintains fixed full-screen dimensions without scrolling
240. HUD overlay elements not affected by scrolling on other screens
241. word-list package successfully integrated and loaded on app initialization
242. word-list dictionary contains 274,000+ words
243. Words from word-list categorized by difficulty based on length
244. Words from word-list assigned to categories (general, tech, finance, medical)
245. Missile spawning selects words from word-list dictionary
246. Word selection filters by difficulty and category correctly
247. Custom word banks override word-list when selected
248. Spaced repetition applies to words from word-list dictionary
249. Speedrun mode uses fixed sequence from word-list with seeded random
250. Multiplayer mode uses word-list with shared seed for deterministic word selection
251. Vocabulary statistics tracked in database
252. Progress reports display unique words encountered from word-list
253. Vocabulary breadth metrics calculated and displayed
254. word-list dictionary cached locally for offline access
255. Word categorization performs efficiently during initialization
256. Word lookup from word-list optimized with indexing
257. Fallback to full word-list dictionary when filtered pool empty
258. Error handling for word-list package load failure
259. Deduplication of words from word-list during initialization
260. Vocabulary milestone events tracked in analytics
261. 3D scene background uses high-resolution star field skybox texture
262. Volumetric fog effect creates depth perception in 3D space
263. Ambient lighting provides base illumination for 3D scene
264. Directional light simulates distant star with correct position and intensity
265. Point lights follow ships and illuminate surrounding 3D space
266. Shadow mapping enabled with soft shadows
267. PBR materials use correct metalness and roughness values
268. Emissive materials glow with correct intensity
269. Particle systems emit continuously for engine trails
270. Explosion particle systems burst on collision with correct colors
271. Spark particle systems emit on ship damage
272. All 3D models use appropriate scale for visual consistency
273. 3D models have correct pivot points for rotation
274. 3D collision detection uses appropriate bounding sphere radius
275. Tournament Hub Screen displays list of active and upcoming tournaments
276. Tournament Hub Screen filters tournaments by status, mode, entry fee, prize pool, start date
277. Tournament Hub Screen search functionality finds tournaments by name or organizer
278. Tournament Hub Screen featured tournaments carousel displays and auto-rotates
279. Tournament Hub Screen My Tournaments section shows registered and created tournaments
280. Tournament Creation Screen allows organizer to configure all tournament settings including Story Mode
281. Tournament Creation Screen validates all required fields before submission
282. Tournament Creation Screen bracket preview updates dynamically
283. Tournament Creation Screen submits tournament creation successfully
284. Tournament Details Screen displays comprehensive tournament information including story title
285. Tournament Details Screen shows participant list with registration status
286. Tournament Details Screen displays match schedule with date/time
287. Tournament Details Screen bracket visualization shows tournament structure
288. Tournament Details Screen registration button allows user to join tournament
289. Tournament Details Screen unregister button allows user to leave before start
290. Tournament Details Screen Start Tournament button (organizer only) begins tournament
291. Tournament Details Screen real-time updates for participant changes
292. Tournament Bracket View displays bracket structure correctly
293. Tournament Bracket View shows match nodes with player names and scores
294. Tournament Bracket View highlights current user's matches
295. Tournament Bracket View highlights active matches in progress
296. Tournament Bracket View auto-scrolls to current user's next match
297. Tournament Bracket View zoom and pan controls work correctly
298. Tournament Bracket View real-time updates as matches complete
299. Tournament Match Lobby displays match information correctly including story title
300. Tournament Match Lobby shows both players with readiness status
301. Tournament Match Lobby Ready button toggles player ready state
302. Tournament Match Lobby match starts automatically when both players ready
303. Tournament Match Lobby countdown timer displays before match start
304. Tournament Match Lobby chat section allows player communication
305. Tournament Match Lobby Cancel button returns to Tournament Details Screen
306. Tournament matches use shared seed for deterministic missile sequence
307. Tournament matches use standardized game settings
308. Tournament matches disable adaptive difficulty for fairness
309. Tournament matches disable spaced repetition for fairness
310. Tournament match result submitted to server on completion
311. Tournament match result validated by server
312. Tournament match winner advances to next round in bracket
313. Tournament match loser eliminated or moves to Losers Bracket
314. Tournament bracket advancement generates next round matches
315. Tournament completion calculates final standings
316. Tournament completion distributes prizes based on settings
317. Tournament completion publishes results on Tournament Details Screen
318. Tournament notifications sent for registration, start, match schedule, results
319. Tournament data persisted to database throughout lifecycle
320. Tournament match forfeit awards win to opponent
321. Tournament match disconnect handling awards win to remaining player
322. Tournament analytics tracked for participation and performance
323. Tournament leaderboard displays tournament winners
324. Tournament Hub accessible from Mode Selection Screen
325. Return to Tournament button on Game Over Screen navigates to Tournament Details
326. Tournament match info displayed in HUD during tournament matches
327. Exit button in tournament match forfeits match and awards win to opponent
328. Tournament system handles edge cases correctly as specified
329. Tournament bracket generation handles insufficient participants
330. Tournament match scheduling avoids conflicts
331. Tournament prize distribution executes successfully
332. Tournament organizer can manage tournament settings
333. Tournament participants receive timely notifications
334. Tournament data backup and recovery functional
335. Story Selection Screen displays list of available stories
336. Story Selection Screen filters stories by duration, difficulty, genre
337. Story Selection Screen displays story cards with title, synopsis, metadata
338. Story Selection Screen Play button starts Story Mode with selected story
339. Story Mode card displays on Mode Selection Screen
340. Story Mode card navigates to Story Selection Screen on click
341. Story Mode game initializes with selected story data
342. AI narration audio plays synchronized with game loop
343. Narrative text displays in HUD panel during Story Mode
344. Current sentence highlighted in bright cyan in narrative panel
345. Narrative text scrolls automatically as narration progresses
346. Story progress bar displays below narrative panel
347. Story progress bar updates as story progresses
348. Missiles spawn from narrative text in sequence
349. Missile spawning synchronized with narration pacing
350. Story Mode uses Zen-like rules (no loss condition)
351. Story Mode ship health cosmetic only
352. Story Mode combo does not reset on missed words
353. Story Mode score multiplier 1.3 applied correctly
354. Story Mode completion bonus awarded for 100% accuracy
355. Story Mode completion percentage calculated correctly
356. Story Mode session ends when all sentences narrated
357. Story Mode allows early exit via exit button with score saved
358. Story Mode partial completion tracked and displayed
359. Story Mode Game Over Screen displays completion percentage
360. Story Mode Game Over Screen displays narrative summary
361. Story Mode Game Over Screen Return to Story Selection button works
362. Story Mode leaderboard displays story title and completion percentage with one entry per player
363. Story Mode leaderboard ranks by score descending
364. Story Mode leaderboard accessible from Leaderboard View
365. Story Mode tournament support functional
366. Story Mode tournament uses same story for all participants
367. Story Mode tournament shared seed ensures synchronized narration
368. Story Mode tournament match displays story title in lobby
369. Story Mode replay includes narration audio
370. Story Mode replay displays narrative text during playback
371. Story Mode replay viewer shows story progress
372. Story Mode analytics events tracked (story_started, story_completed, story_progress)
373. Story Mode session data persisted to database
374. Story Mode completion data stored in story_completions table
375. Story Mode stories cached locally for offline play
376. Story Mode offline play functional with cached stories
377. Story Mode narration audio pre-generated and cached
378. Story Mode narration speed adjustable
379. Story Mode narrator voice configurable per story
380. Story Mode timing data used for missile spawn synchronization
381. Story Mode word extraction from narrative text functional
382. Story Mode missile difficulty assigned based on word length
383. Story Mode spawn rate adjusted to match narration speed
384. Story Mode adaptive difficulty disabled
385. Story Mode spaced repetition disabled
386. Story Mode camera shake disabled
387. Story Mode exit button pauses narration audio and saves score
388. Story Mode exit confirmation resumes narration on No
389. Story Mode friend activity feed displays story completions
390. Story Mode vocabulary stats tracked separately
391. **Session data saved to database when user exits game via exit button**
392. **Score submitted to leaderboard when user exits game via exit button**
393. **Exit button displays confirmation message indicating progress has been saved**
394. **Game over screen displays after exit button confirmation with saved score**
395. **Leaderboard entry created or updated when user exits game early**
396. **Session analytics events flushed immediately on exit button click**
397. **Word stats updated in database when user exits game early**
398. **Personal best updated if applicable when user exits game early**
399. **Rank calculation triggered when user exits game early**
400. **All game modes save score when user exits via exit button**

## 7. Out of Scope for This Release

- Password reset functionality
- Email verification
- Social login integration
- Voice chat in multiplayer
- Daily challenges
- Seasonal events
- In-game purchases or monetization
- Accessibility features (screen reader support, colorblind modes)
- Localization and internationalization
- Settings panel for customization
- Ad integration
- Customization options (unlockable ship skins, missile effects, backgrounds)
- Advanced AI-powered typing analysis
- Integration with external typing test platforms
- Mobile-optimized touch controls
- Keyboard layout customization
- Macro detection and prevention
- Anti-cheat system for competitive play
- Streaming integration (Twitch, YouTube)
- Discord bot integration
- API for third-party integrations
- White-label licensing for B2B customers
- Database sharding for horizontal scaling
- Multi-region database replication
- Advanced caching strategies
- Real-time database change notifications
- Power-ups or special abilities
- Different spaceship types with unique abilities
- Environmental hazards (asteroids, black holes) as interactive gameplay elements
- Co-op multiplayer mode
- Team-based multiplayer battles
- Spectator mode for multiplayer matches
- In-game chat system
- Emote system
- Achievement badges and unlockables
- Player profile customization
- Clan or guild system
- Global events or world bosses
- Procedurally generated levels
- Sound effects and background music
- Voice acting
- Cinematic cutscenes
- Word definitions display on Game Over screen
- Advanced vocabulary learning features
- Integration with language learning platforms
- Multiple language support for word-list
- Custom word-list sources beyond npm package
- Word pronunciation audio
- Etymology or word origin information
- Synonym and antonym suggestions
- Contextual word usage examples
- VR or AR support for immersive 3D experience
- Advanced post-processing effects (bloom, depth of field, motion blur)
- Physically-based sky and atmosphere rendering
- Procedural planet generation
- Advanced particle effects (fire, smoke, debris)
- Cloth simulation for ship flags or banners
- Destructible 3D environments
- Advanced AI for enemy ship behavior
- Dynamic weather effects in space
- Realistic physics simulation for ship movement
- Advanced camera modes (cinematic, follow, orbit)
- Level of detail (LOD) system for distant objects
- Occlusion culling for performance optimization
- Instanced rendering for large numbers of objects
- Custom shaders for advanced visual effects
- Ray tracing or path tracing for realistic lighting
- Global illumination for indirect lighting
- Subsurface scattering for translucent materials
- Volumetric lighting for god rays and light shafts
- Screen space reflections for realistic reflections
- Temporal anti-aliasing for smoother edges
- HDR rendering and tone mapping customization
- Historical leaderboard data visualization
- Leaderboard filtering by country or region
- Leaderboard filtering by friend list only
- Leaderboard export functionality
- Leaderboard API for third-party access
- Tournament spectator mode
- Tournament live streaming integration
- Tournament prize pool crowdfunding
- Tournament sponsorship system
- Tournament referee or admin tools
- Tournament appeal or dispute resolution system
- Tournament statistics and analytics dashboard for organizers
- Tournament replay analysis tools
- Tournament coaching or mentoring features
- Tournament team registration for team-based tournaments
- Story Mode editor for user-generated stories
- Story Mode community story sharing platform
- Story Mode branching narratives with player choices
- Story Mode multiple endings based on performance
- Story Mode character customization
- Story Mode voice actor selection
- Story Mode background music and sound effects
- Story Mode visual novel style cutscenes
- Story Mode achievement system
- Story Mode difficulty scaling based on player performance
- Story Mode adaptive narration speed
- Story Mode interactive dialogue choices
- Story Mode story progression tracking across multiple sessions
- Story Mode story recommendations based on player preferences
- Story Mode story rating and review system