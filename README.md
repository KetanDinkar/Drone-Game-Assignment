
# Drone vs Enemy Combat Simulation (Prototype)
 
A Unity prototype where the player pilots a drone armed with missiles against
patrolling enemy soldiers. Enemies patrol randomly using NavMesh, detect the
drone when it enters their range, stop and aim (with IK-driven hand
targeting) at the drone, and fire lasers at it. The drone can auto-target and
missile-strike attacking enemies, destroying them with a hit reaction and
respawn cycle.
 
## 📁 Build / Unity Project (Google Drive)
[Drone vs Enemy Combat Simulation — Build & Project Files](https://drive.google.com/drive/folders/1Dkz92IgcEzD-AmtbI07ujPe7DF8u3Z11?usp=sharing)
 
> Hosted on Drive since the packaged build exceeds GitHub's file size limits.
> This repo contains the full Unity source project.
 
## 🎥 Demo Video
[Add your demo video link here]
 
## 📦 GitHub Repository
[Add your repository link here]
 
---
 
## 🎮 Controls
 
| Input | Action |
|---|---|
| **W / A / S / D** | Move drone forward / left / backward / right (relative to drone's facing) |
| **Q** | Descend |
| **E** | Ascend |
| **Spacebar** | Fire missile |
| **Start Button (UI)** | Begins the game (game is paused on load until pressed) |
| **Restart Button (UI)** | Restarts the game after the drone is destroyed |
| **Exit Button (UI)** | Quits the application |
 
---
 
## ✅ Features Implemented
 
### 🚁 Player Drone
- Free-flight movement on WASD + Q/E, moved relative to the drone's own
  orientation (`TransformDirection`).
- Movement is clamped to a defined X/Y/Z bounding box (`xMin/xMax`,
  `yMin/yMax`, `zMin/zMax`) so the drone can't fly outside the play area.
- Simple collision avoidance using `Physics.BoxCast` against an
  `obstacleLayer` so the drone doesn't fly through obstacles.
- Fires a missile on **Spacebar**, with a cooldown between shots
  (`missileCooldown`) and a max lifetime per missile (`missileLifetime`).
- Drone rotor/fan animation via continuous local rotation (`fanrotation.cs`)
  for a simple spinning-propeller visual.
- Audio: hit sound and "falling/destroyed" sound played through the drone's
  `AudioSource`; missile-fire sound played on each shot.
### 💣 Missile System
- Missile is spawned at a `missileFirePoint`, then a `MissileBehavior`
  component is attached at runtime to control its flight.
- **Auto-targeting:** on fire, the drone scans all active `DRONEDETECTOR`
  components (one per enemy) and picks the **nearest enemy that currently
  has the drone inside its detection zone** (i.e. an enemy actively
  attacking), aiming at that enemy's `SHOOT` child transform.
- If **no enemy is currently attacking**, the missile has no target and
  simply flies forward with no homing.
- If **multiple enemies are attacking at once**, the missile locks onto
  whichever attacking enemy is closest to the drone.
- On hitting an enemy (trigger with tag `ROCKET`), `MESSILEHITTRACKER`
  destroys the missile and the enemy in one hit, and reports the kill back
  to the enemy manager.
### 🤖 Enemy AI
- Enemies are spawned at designated `enemy_moving_point` waypoints and
  patrol autonomously using `NavMeshAgent` + `NavMesh.SamplePosition`,
  wandering to random points within a `wanderRadius` at randomized
  intervals — not a fixed back-and-forth waypoint loop.
- **Detection:** each enemy has a trigger collider (`DRONEDETECTOR`) that
  flags `isDroneInside = true` when the drone enters range.
- **Attack behavior:** once the drone is detected, the enemy:
  - Stops moving and switches from a "walk" animation to an "idle/aim"
    animation.
  - Rotates smoothly to face the drone.
  - Drives an IK hand target toward the drone's position (so the arm/weapon
    visually tracks the drone) — gives the aiming a natural look instead of
    a snap-to-target.
  - Fires laser projectiles at the drone at a fixed fire rate
    (`fireRate`), from a dedicated `LaserFirePoint`.
  - Resumes patrolling automatically once the drone leaves detection range.
- **Death & respawn:** when an enemy is destroyed by a missile, a blast
  particle effect and sound play, the enemy is removed, and after a
  `respawnDelay` a new enemy is spawned back at that same waypoint —
  patrolling starts again with a small random delay so multiple enemies
  don't all move in perfect sync.
### ❤️ Drone Health & Damage
- Drone has a UI `Slider` representing health, starting full.
- Each laser hit reduces health by a fixed amount, plays a hit sound and a
  small blast/spark particle effect.
- When health reaches zero: the drone is disabled (movement + firing turned
  off), all colliders are disabled, and the drone plays a falling animation
  (interpolated fall down to `yMin`) followed by a "crash" sound, after
  which the Restart UI is shown.
### 🔁 Game Flow / UI
- **Start screen:** game loads paused (`Time.timeScale = 0`); Start and
  Exit buttons are shown until the player presses Start, which resumes
  time and hides the menu.
- **Restart screen:** appears automatically once the drone is destroyed.
  Restarting resets the drone to its start position/rotation, resets its
  health and re-enables its colliders and controls, clears remaining
  lasers/enemies, and respawns all enemies fresh.
- **Exit:** quits the application from either the start or restart screen.
---
 
## 🗂️ Key Scripts
 
| Script | Responsibility |
|---|---|
| `DRONEMOVERANDFIRE.cs` | Drone movement, bounds/collision, missile firing, auto-target selection, missile homing behavior |
| `ENEMYMOVERANDATTACKS.cs` | Enemy spawning, NavMesh patrol/wander, detection response, IK aim, laser firing, death + respawn |
| `DRONEDETECTOR.cs` | Trigger zone on each enemy that flags when the drone is in range |
| `MESSILEHITTRACKER.cs` | Detects missile ("ROCKET" tag) hits on an enemy and reports the kill |
| `LASEATTACKDETECTED.cs` | Handles laser hits on the drone, health/UI updates, death sequence |
| `Restart game.cs` | Restart/Exit UI logic and full game-state reset |
| `start button.cs` | Start-menu pause/unpause logic |
| `fanrotation.cs` | Continuous rotor-fan spin for the drone model |
 
---
 
## 🛠️ Setup / How to Run
1. Open the project in Unity (see `ProjectSettings` for the exact version
   used).
2. Ensure a `NavMesh` is baked in the scene (Window → AI → Navigation) so
   enemies can patrol.
3. Confirm the following tags exist and are assigned: `DRONE` (on the
   drone), `LASER` (on enemy laser prefab), `ROCKET` (on the missile
   prefab).
4. Press Play — the game starts paused on the Start screen; press **Start**
   to begin.
---
 
## ⚠️ Known Bugs / Limitations
- The task brief asks for generic destroyable **"Target"**-tagged dummy
  objects in addition to enemies; the current build only implements
  missile destruction against enemies (`ROCKET` vs enemy hit-tracker) —
  static dummy targets are not yet wired up.
- When no enemy is currently attacking, the fired missile is spawned using
  the missile **prefab's own fixed rotation** rather than the drone's
  current facing direction, so an "untargeted" missile always launches in
  the same world-space direction rather than straight ahead from wherever
  the drone is currently pointed.
- Enemy patrol is fully randomized wandering within a radius rather than a
  fixed waypoint-to-waypoint patrol route.
- Camera-follow behavior is expected to be set up in-scene (e.g. as a child
  camera or Cinemachine rig) and is not handled by a dedicated script in
  this codebase.
- No explicit game-over/score/UI feedback beyond health bar and
  restart/exit buttons.
---
 
## 📸 Screenshots
 
| | |
|---|---|
| ![Start Screen](Screenshots/01_start_screen.jpg) **Start screen** — drone parked in the warehouse, enemies idle in the background, full health, "START" button paused (`Time.timeScale = 0`). | ![Full Health Gameplay](Screenshots/02_gameplay_full_health.jpg) **Gameplay begins** — after Start is pressed, multiple enemies are visible patrolling/standing near the drone's spawn area. |
| ![Enemy Detected](Screenshots/03_enemy_detected_laser.jpg) **Enemy engaging** — an enemy has detected the drone and is firing a red laser bolt at it; health has started dropping. | ![Missile Fired](Screenshots/04_missile_fired_at_enemy.jpg) **Missile fired** — the drone's homing missile is mid-flight toward the nearest attacking enemy, whose health bar is now visible and depleting. |
| ![Under Heavy Fire](Screenshots/05_drone_under_heavy_fire.jpg) **Taking heavy fire** — several enemies converge and fire on the drone at once; health is critically low and a hit-reaction effect flashes on the drone. | ![Low Health](Screenshots/06_low_health_multiple_enemies.jpg) **Low health, multiple attackers** — two enemies actively lasering the drone as health nears zero, shortly before the death/fall sequence triggers. |
 
> Screenshot files are in the `Screenshots/` folder of this repo. If you rename or move them, update the image paths above to match.
 


















