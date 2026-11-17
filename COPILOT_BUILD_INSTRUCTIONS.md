# ECHOVERSE — GitHub Copilot Build Instructions

> **Purpose:** Paste this file into a GitHub repository as `COPILOT_BUILD_INSTRUCTIONS.md`, then ask GitHub Copilot Agent to implement it. This is a development specification, **not executable game code**. Build the playable MVP first; do not attempt all features in one pass.

## 1. Game vision

**Title:** ECHOVERSE: The World That Remembers  
**Genre:** Browser-based 3D puzzle adventure / mystery / creative sandbox  
**Core hook:** The player's previous actions can be recorded and replayed as interactive Echo Clones. Multiple versions of the player cooperate to solve spatial puzzles. Later, players unlock reality-manipulation abilities.

**Player promise:** "Don't just find a solution. Invent one."

**Target:** Modern desktop browsers first; mobile responsive UI and controls later. Single-player MVP first, multiplayer later.

### Pillars
1. **Discover through interaction:** Short visual hints rather than lengthy tutorials.
2. **Replayable history:** Echo Clones reliably repeat player actions in a deterministic simulation.
3. **Creative solutions:** Accept multiple valid ways to reach puzzle goals.
4. **Readable 3D:** Clear silhouettes, color-coded interactive objects, accessible controls.
5. **Fast browser load:** Keep assets lightweight and performance measurable.

## 2. Technical stack

- Vite + React + TypeScript
- Three.js via `@react-three/fiber` and `@react-three/drei`
- `@react-three/rapier` for physics, where useful
- Zustand for game state
- Vitest + React Testing Library for logic/UI tests
- Playwright for browser smoke tests (optional during initial setup)
- ESLint + Prettier
- No backend required for the MVP; localStorage for local settings and progress

Future integrations (do **not** build until later phases): Supabase for user accounts/levels, Colyseus for multiplayer, Node.js services, Vercel deployment.

## 3. The MVP: "The Room of Three Selves"

Create a complete, playable 3D level featuring:

- A small atmospheric sci-fi chamber with a floor, walls, lighting, and one sealed exit.
- Third-person or first-person player controller with WASD, mouse look, jump (Space), and interact (E). Choose one camera mode and make it polished.
- Three pressure plates, spaced so one player cannot activate all at once.
- A glowing Echo Recorder UI with **Start Recording**, **Stop Recording**, **Play Echo**, **Clear Echoes** controls and keyboard shortcuts.
- Record player movement and plate interactions for up to 10 seconds per Echo.
- Playback produces a translucent, glowing Echo Clone that can hold pressure plates.
- At least **two concurrent Echo Clones** plus the live player.
- Door opens when all three plates are simultaneously occupied by a player or Echo.
- Door leads to a finish trigger, victory overlay, elapsed time, and restart option.
- Tutorial hints, pause menu, reset, and a brief onboarding screen.
- Basic audio toggle and reduced-motion setting.

**Intended discovery:** The player records standing on plate A, plays Echo A, records standing on plate B, plays Echo B, and stands on plate C. All three plates illuminate, the door opens, and the player exits.

### Important design clarification
A clone must replay **recorded player actions**, not generate arbitrary geometry from movement in the MVP. Turning trajectories into physical bridges is a future mechanic with separate constraints. Keep these two systems distinct.

## 4. Architecture and folder layout

Use this proposed structure (adjust only if there is a concrete reason):

```text
src/
  app/
    App.tsx
    styles.css
  game/
    GameCanvas.tsx
    GameLoop.tsx
    constants.ts
    components/
      Chamber.tsx
      Player.tsx
      EchoClone.tsx
      PressurePlate.tsx
      ExitDoor.tsx
      FinishTrigger.tsx
      InteractivePrompt.tsx
    systems/
      recorder.ts
      playback.ts
      plateOccupancy.ts
      puzzleState.ts
      input.ts
      saveProgress.ts
    state/
      useGameStore.ts
    types/
      game.ts
    levels/
      roomOfThreeSelves.ts
  ui/
    HUD.tsx
    MainMenu.tsx
    PauseMenu.tsx
    VictoryScreen.tsx
    SettingsPanel.tsx
  tests/
    recorder.test.ts
    playback.test.ts
    plateOccupancy.test.ts
    puzzleState.test.ts
public/
  audio/
  textures/
```

Use procedural geometry and simple materials at first. Avoid requiring paid models, proprietary assets, external API keys, or copyrighted game content.

## 5. Deterministic Echo recording — engineering requirements

### Simulation
- Use a fixed simulation timestep of **1/60 second** for recording and playback, with an accumulator if render FPS varies.
- Record an ordered sequence of frames containing: `tick`, position, rotation, relevant actions (jump/interact), and optionally velocity.
- Use interpolation for rendering between frames; gameplay interactions must derive from the fixed-tick simulation state.
- Cap each recording at 10 seconds / 600 frames.
- Support at least two active clones, with a configurable limit.
- On replay, clones follow the recorded trajectory exactly. For the MVP, treat clones as **kinematic replay actors** to avoid divergence caused by physics collisions.
- Clones trigger pressure plates but do not block the player or collide with each other.
- A recording's start point is world-space; play it back in the same world location where it was captured.
- Clones loop only if explicitly configured; default to **hold final pose** so standing on a plate remains useful.
- The UI must clearly show recording status, duration, active Echo count, and errors.
- Reset must clear recordings, clones, plate occupancy, and door state.

### Suggested TypeScript contracts

```ts
export type Vec3 = { x: number; y: number; z: number };
export type PlayerAction = 'jump' | 'interact';
export interface EchoFrame {
  tick: number;
  position: Vec3;
  rotationY: number;
  actions: PlayerAction[];
}
export interface EchoRecording {
  id: string;
  createdAt: number;
  frames: EchoFrame[];
  endBehavior: 'hold' | 'loop' | 'despawn';
}
export interface EchoPlayback {
  recordingId: string;
  playbackTick: number;
  isActive: boolean;
}
```

### Plate occupancy
- Each plate has a unique ID, radius, and world position.
- Compute occupancy by comparing player/Echo actor footprints against the plate's trigger region at each fixed tick.
- Each plate is active when **at least one** eligible actor is present.
- Door opens when **all three** plates are active on the same tick.
- Keep door open once solved for the MVP, so it doesn't close on the player.
- Render active/inactive states visibly and provide an accessible textual indicator.

## 6. Controls and UX

| Action | Desktop input |
|---|---|
| Move | WASD / arrow keys |
| Look | Mouse / pointer lock |
| Jump | Space |
| Interact | E |
| Start/stop recording | R |
| Play most recent recording | P |
| Pause / unlock pointer | Escape |
| Restart level | Button in pause menu |

Show the controls before gameplay. Make pointer-lock errors recoverable. Ensure UI buttons are clickable even when pointer lock is active. On mobile, add virtual joystick and touch controls only after desktop MVP is stable.

## 7. Visual and audio direction

**Mood:** Surreal futuristic archive; mysterious but approachable.  
**Palette:** Deep navy and charcoal environment; cyan Echo Clones; amber inactive plate accents; vivid green active plate accents.  
**Art:** Geometric architecture, glowing lines, fog, floating particles (performance permitting).  
**Feedback:** Subtle pulse when recording starts, clear plate activation sound, door-opening animation, victory effect.  
**Accessibility:** High-contrast UI, readable labels, audio mute, reduced-motion mode, no puzzle information conveyed by color alone.

## 8. Seven reality laws — post-MVP roadmap

Build these as separate, testable systems after the Echo MVP works:

1. **Memory:** Echo Clones replay history. Advanced extension: convert recorded paths into temporary solid geometry with limits on duration, volume, and overlap.
2. **Gravity:** Rotate gravity for tagged objects or bounded zones, not the entire simulation indiscriminately.
3. **Time:** Rewind selected objects using bounded state histories and explicit conflict rules.
4. **Perspective:** Resize tagged objects based on controlled camera projection, with min/max sizes and anti-exploit constraints.
5. **Sound:** Trigger responsive mechanisms with simple frequency or rhythm patterns; do not require microphone access.
6. **Connection:** Link two tagged objects through a configurable motion constraint.
7. **Paradox:** Temporarily override one defined puzzle constraint; show countdown and reset safely.

**Important:** Introduce one law per milestone with 3–5 handcrafted puzzles and tests. Do not build all seven at once.

## 9. Game worlds and story

**Setting:** The Archive, a simulated universe of impossible chambers, floating cities, and structures that defy physics.

**Narrative:** The mysterious Architect leaves recordings that initially seem instructional. Later, the player learns that the Architect may be a future version of themselves. Solving puzzles trains the Archive to reconstruct a collapsing universe.

**Ending concept:** A blank white chamber presents: "You have learned every rule. Now create one." The player combines learned mechanics to construct a valid exit. This is a future narrative design goal, not MVP scope.

## 10. Planned game modes

- **Solo Campaign:** Story-driven progression through puzzle chambers.
- **Reality Race:** Two players solve equivalent rooms; fastest wins.
- **Echo Duel:** Competitive puzzles using recorded clones.
- **Paradox Escape:** Four-player cooperative puzzle rooms.
- **Puzzle Architect:** In-browser drag-and-drop room editor.
- **Impossible Rooms:** Community-published levels, ratings, and difficulty.
- **Daily Paradox:** A shared daily puzzle and leaderboard.

For creator levels, require a recorded successful completion before publishing. Preserve the verified solution privately; players may find other valid solutions. Server-side validation and anti-cheat are required before competitive leaderboards or paid creator features.

## 11. Monetization principles (future only)

Keep core puzzle-solving abilities free and competitive modes fair. Potential revenue: cosmetic skins, Echo visual effects, room themes, optional season passes, premium creator collections, and private servers. Do not implement payment processing in the MVP.

## 12. Build milestones and completion criteria

### Milestone A — Project scaffold
- Create Vite/React/TypeScript app and install dependencies.
- Add scripts: `dev`, `build`, `preview`, `lint`, `test`.
- Add a concise README with local setup and controls.
- Confirm `npm install`, `npm run dev`, and `npm run build` work.

### Milestone B — 3D chamber and movement
- Render a chamber, player, floor, camera, and lights.
- Add movement and jump; prevent walking through walls.
- Add pause and restart.

### Milestone C — Recording and playback
- Implement fixed-tick recorder and kinematic playback.
- Show translucent Echo Clones.
- Enforce 10-second cap and 2-clone limit.
- Add unit tests for capture/playback edge cases.

### Milestone D — Puzzle and victory
- Add 3 pressure plates and a locked exit.
- Count live player and clones for plate activation.
- Open door when all 3 are occupied simultaneously.
- Add finish trigger, victory UI, hints, and audio toggle.
- Add tests for plate logic and reset.

### Milestone E — Polish and deployment
- Improve visual effects, accessibility, responsive HUD, and performance.
- Run build, lint, and tests; fix failures.
- Document Vercel deployment steps.

### Definition of done for MVP
A fresh user can open the browser game, understand controls, create two Echo Clones, hold all three plates simultaneously, open the exit, finish the level, and restart **without console errors**. `npm run build`, `npm run lint`, and `npm run test` must pass.

## 13. Instructions for GitHub Copilot Agent

Act as a senior TypeScript game developer. Read this entire document, then:

1. Inspect the repository before making changes. If it is empty, scaffold the project.
2. **Implement Milestone A only in your first pass.** Do not try to build the whole game in one response.
3. Create actual working source files; do not respond with only pseudocode.
4. Explain which files were created and which commands to run.
5. Run available checks and fix errors before reporting completion.
6. Do not add unnecessary dependencies, services, or secrets.
7. Ask for approval before starting the next milestone.
8. For each milestone, add or update tests and README documentation.
9. If a design decision is ambiguous, choose the simplest stable approach and document it.
10. Prioritize a working game over ambitious untested mechanics.

### Copy/paste starter prompt for Copilot Chat

> Read `COPILOT_BUILD_INSTRUCTIONS.md` completely. You are implementing ECHOVERSE, a browser-based 3D puzzle game. Start with **Milestone A only**: scaffold Vite + React + TypeScript, install React Three Fiber, Drei, Rapier, Zustand, and test/lint tooling; create the folder structure, package scripts, and README. Create real files and run the build. Tell me exactly how to start the development server. Do not implement later milestones yet.

### Follow-up prompts

**After A passes:** "Implement Milestone B from `COPILOT_BUILD_INSTRUCTIONS.md`. Build a playable 3D chamber with working movement, collision, camera, pause, and reset. Test and fix errors."

**After B passes:** "Implement Milestone C. Use fixed-tick recording and deterministic kinematic Echo playback. Add two simultaneous clones, UI controls, and unit tests."

**After C passes:** "Implement Milestone D. Add three pressure plates, occupancy logic, exit door, victory trigger, hints, and tests. Verify the full puzzle can be completed."

**After D passes:** "Implement Milestone E. Improve performance, visual clarity, accessibility, and deployment documentation. Run all checks."

## 14. Known risks and guardrails

- Physics replay can diverge: replay recorded transforms for Echo actors rather than re-simulating physics inputs.
- Clones may overlap: allow overlaps but use independent occupancy checks.
- Door may flicker: latch solved state once all plates activate.
- Browser performance may suffer: cap clones, particles, dynamic lights, and shadow quality.
- Pointer lock can block UI: release pointer lock on pause/menu.
- User-generated levels may be unsolvable: require creator proof-of-solution and eventually validate server-side.
- Multiplayer synchronization is hard: defer it until single-player deterministic logic is stable.

---

**Repository guidance:** Commit each milestone separately. This document is the product specification and implementation plan. The executable game is the code Copilot creates in the repository.
