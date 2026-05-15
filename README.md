# Brick Breaker — Java Swing
--------------------------

> A 2D arcade game built with Java Swing, exploring event-driven game loop design, AABB collision detection, and entity state management — documented with architectural honesty.

---

## About The Project

This project is a desktop implementation of the classic Breakout/Brick Breaker arcade game, written entirely in Java using the Swing GUI toolkit. It was built to explore the mechanics of event-driven programming — specifically how a game loop, physics updates, and rendering can be orchestrated within a single-threaded UI framework.

The codebase is intentionally kept small and inspectable. It demonstrates how javax.swing.Timer can drive a fixed-interval simulation tick, how AABB (Axis-Aligned Bounding Box) collision detection works in practice, and how entity state (bricks, ball, power-ups) can be managed without a dedicated game engine.

This repository is documented with architectural transparency — including known bugs, design tradeoffs, and what a production-quality redesign would look like. It is intended as a portfolio artifact that demonstrates the ability to critically evaluate one's own work, not just ship it.

---

## Project Status

**Archived Learning Project** — Feature-complete for its intended scope. No active development. Documented for portfolio and interview purposes.

---

## Why I Built This

The primary motivation was to understand how real-time systems work at a low level — specifically:

*   How a game loop drives consistent frame updates without blocking the UI thread (and what breaks when it does)
    
*   How collision detection works geometrically, and where naive implementations fail
    
*   How entity lifecycle (creation, state mutation, soft deletion) can be managed without a full ECS framework
    
*   How Java's javax.sound.sampled API handles audio playback in a real-time context
    

A secondary goal was to build something inspectable enough to discuss deeply in technical interviews — understanding not just what the code does, but why certain decisions were made and what the correct alternatives are.

---

## Features

#### Core Gameplay

*   Brick grid destruction with score tracking
    
*   Ball physics with direction-reversal collision response
    
*   Paddle control via keyboard (left/right arrow keys)
    
*   Win and lose state detection with restart capability
    
*   Pause/resume toggle (P key)
    

#### Engineering Features

*   javax.swing.Timer-based simulation tick at ~8ms intervals
    
*   AABB collision detection using java.awt.Rectangle.intersects()
    
*   Soft-delete pattern for power-up entity lifecycle management
    
*   Labeled break (break A) for single-brick-per-tick collision resolution
    
*   Dynamic power-up spawning with 10% probability on brick destruction
    

#### Audio

*   Sound effect playback on ball collision via javax.sound.sampled.Clip
    
*   Rewindable clip with setFramePosition(0) for rapid replays
    
---
## Tech Stack

| Layer | Technology | Reason |
|---|---|---|
| Language | Java 11+ | Platform target; standard library completeness |
| UI / Rendering | Java Swing (`JPanel`, `JFrame`) | No external dependencies; built-in 2D graphics |
| Game Loop | `javax.swing.Timer` | EDT-integrated tick dispatch; avoids manual thread synchronization |
| Audio | `javax.sound.sampled` | Built-in Java audio; no third-party dependency |
| Build | Manual (Eclipse/IntelliJ project) | No Maven/Gradle — noted as technical debt |
| Module System | Java 9 JPMS (`module-info.java`) | Declares `requires java.desktop` for Swing/AWT access |

---

## Architecture

```text
┌─────────────────────────────────────────────────┐
│                    JFrame                      │
│  ┌───────────────────────────────────────────┐ │
│  │          GameComponents (JPanel)          │ │
│  │                                           │ │
│  │  ┌─────────────┐   ┌──────────────────┐   │ │
│  │  │ Map         │   │ PowerUps[]       │   │ │
│  │  │ (brick grid)│   │ (ArrayList)      │   │ │
│  │  └─────────────┘   └──────────────────┘   │ │
│  │                                           │ │
│  │  ┌─────────────────────────────────────┐  │ │
│  │  │ javax.swing.Timer (8ms tick)        │  │ │
│  │  │ → actionPerformed()                 │  │ │
│  │  │   ├─ Paddle collision (AABB)        │  │ │
│  │  │   ├─ Brick scan + collision         │  │ │
│  │  │   ├─ PowerUp physics + collection   │  │ │
│  │  │   ├─ Ball position update           │  │ │
│  │  │   └─ repaint()                      │  │ │
│  │  └─────────────────────────────────────┘  │ │
│  │                                           │ │
│  │  KeyListener → playerX mutation           │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

---

### Simulation Tick Lifecycle

```text
Timer fires (every 8ms)
   └── actionPerformed()
        ├── Ball vs Paddle
        │     → AABB → reverse ballYdir
        │
        ├── Ball vs Bricks
        │     → O(R×C) scan
        │     → setBrickValue(0)
        │     → score++
        │     → maybe spawn PowerUp
        │
        ├── PowerUps
        │     → y += 1
        │     → check paddle intercept
        │     → check out-of-bounds
        │
        ├── Ball movement
        │     → ballposX += ballXdir
        │     → ballposY += ballYdir
        │
        ├── Wall bounce
        │     → clamp and reverse on boundary
        │
        └── repaint()
              → triggers paint()
              → full scene redraw
```

---

### Folder Structure

```text
Brickbreaker/
├── src/
│   ├── module-info.java
│   │   # JPMS module declaration
│   │
│   └── game/
│       ├── Main.java
│       │   # Entry point — JFrame bootstrap
│       │
│       ├── GameComponents.java
│       │   # Core game class — loop, input, physics, rendering
│       │
│       ├── Map.java
│       │   # Brick grid data structure and renderer
│       │
│       └── PowerUps.java
│           # Power-up entity
│
├── brickbreaker.png
│   # Window icon asset
│
├── Sound.wav
│   # Collision sound effect
│
└── README.md
```

---

## Installation & Running

### Prerequisites

- Java 11 or higher

### Clone the Repository

```bash
git clone https://github.com/Heramb1221/brick-breaker-java.git
cd brick-breaker-java
```

### Compile

```bash
javac --module-source-path src -d out $(find src -name "*.java")
```

### Run

```bash
java --module-path out -m Brickbreaker/game.Main
```

> **Note:** `Sound.wav` and `brickbreaker.png` must be present in the working directory at runtime. These are currently loaded via relative file paths — a known limitation documented under [Technical Debt](#technical-debt).

---

## Controls

| Key | Action |
|---|---|
| ← Left Arrow | Move paddle left |
| → Right Arrow | Move paddle right |
| P | Pause / Resume |
| Enter | Start game / Restart |

---

## Power-Up System

| Type | Color | Effect | Spawn Condition |
|---|---|---|---|
| Type 1 (Width) | Orange | Increases paddle width from `100px` to `150px` | 10% chance on brick destruction |
| Type 2 (Speed) | Green | Doubles ball velocity | Defined but intentionally disabled |

---

## Screenshots
<img width="986" height="741" alt="Screenshot 2026-05-15 223014" src="https://github.com/user-attachments/assets/32a61c17-1efe-4f54-9fc7-d86512b45c72" />

---

<img width="993" height="745" alt="Screenshot 2026-05-15 223025" src="https://github.com/user-attachments/assets/77e13a4a-252b-4d2c-91ea-c2d2cae9bce1" />

---

<img width="982" height="744" alt="Screenshot 2026-05-15 223035" src="https://github.com/user-attachments/assets/f84bd112-86a0-4db9-a80e-50a82c601817" />

---

## Performance Considerations

- **Tick rate:** `8ms` timer interval targets ~125 FPS. Actual throughput depends on EDT load.

- **Brick scan:** `O(R × C)` per tick. At `3×8 = 24` bricks, this is negligible. Not designed for large grids.

- **Rendering:** Full scene repaint every tick — no dirty-rectangle optimization. Suitable for this scale.

- **Power-up list:** Soft-delete pattern avoids `ConcurrentModificationException`.  
  The list is bounded by `powerUps.clear()` on restart. Mid-game accumulation of invisible entries is a minor memory consideration.

- **Object allocation:** `new Rectangle()` is allocated in the hot collision path per tick. Reusing pre-allocated instances would reduce GC pressure at higher tick rates.

---

## Known Issues

| Issue | Severity | Detail |
|---|---|---|
| Restart state inconsistency | Medium | Initial game uses `Map(3,8)` with `24` bricks; `restartGame()` uses `Map(3,7)` with `21` bricks — layout changes after restart |
| `paint()` vs `paintComponent()` | Medium | Overriding `paint()` bypasses Swing double-buffer optimization, risking visible tearing |
| `g.dispose()` on passed `Graphics` | Medium | Disposing Swing-managed `Graphics` may corrupt rendering |
| No delta-time physics | Low | Ball speed becomes frame-rate dependent |
| Type-2 power-up disabled | Low | Velocity doubling causes tunneling through bricks |
| Asset loading via relative path | Low | `Sound.wav` and icon fail if working directory changes |
| Clip resource not closed | Low | `javax.sound.sampled.Clip` is never explicitly closed |

---

## Contributing

Suggestions, improvements, and pull requests are welcome.

```bash
# Fork repository

# Create feature branch
git checkout -b improvement/your-feature

# Commit changes
git commit -m "Add: feature description"

# Push branch
git push origin improvement/your-feature
```
---

## Technical Debt

- **No build system:** No Maven or Gradle configuration. Manual compilation is fragile and non-reproducible.

- **God Object pattern:** `GameComponents` handles:
  - Input
  - Physics
  - Collision detection
  - Entity management
  - Rendering
  - Audio

  This limits extensibility.

- **Magic numbers:** Coordinates such as `550`, `570`, `784`, `80`, and `50` are hardcoded rather than derived from constants or layout dimensions.

- **No testing:** No unit or integration tests for collision logic, power-up behavior, or state transitions.

- **Public fields on data classes:** `Map` and `PowerUps` expose mutable public state without encapsulation.

---

## Tradeoffs & Limitations

### EDT-Based Game Loop vs Dedicated Thread

Running the game loop via `javax.swing.Timer` keeps all state mutation on the EDT, avoiding synchronization complexity.

#### Tradeoff

Any update taking longer than `8ms` delays:

- Rendering
- Input processing
- Timer execution

For this scale, this is acceptable.

### Production Architecture

```text
Dedicated update thread
        ↓
Game state mutation
        ↓
SwingUtilities.invokeLater()
        ↓
Render on EDT
```

---

### AABB Collision vs Circle Collision

The ball is rendered visually as an oval but treated mathematically as a rectangle.

#### Result

Small collision inaccuracies occur during angled impacts.

### Accurate Alternative

True circle-rectangle collision:

- Compute nearest point on rectangle
- Measure distance to circle center

More accurate, but unnecessary for this scale.

---

### Immediate-Mode Rendering vs Scene Graph

Every frame redraws the full scene.

This is acceptable for:

- 24 bricks
- 1 paddle
- 1 ball

A retained scene graph or dirty-region rendering would become necessary at larger scales.

---

## Scalability Discussion

This architecture is intentionally scoped to a single-player desktop game.

There is no networked or multiplayer scalability model.

### Current Constraints

| Component | Scalability Concern | Future Optimization |
|---|---|---|
| Brick grid | `O(R×C)` collision scan per tick | Spatial partitioning / quadtree |
| Power-up list | Linear growth per session | ECS-style entity pooling |
| Rendering | Full repaint every frame | Dirty-rectangle rendering |

---

## What I Learned

- **Swing EDT threading model:** Why blocking the EDT harms rendering and responsiveness.

- **Swing painting contract:** The distinction between `paint()` and `paintComponent()`, and how Swing double buffering works.

- **AABB collision systems:** Practical tradeoffs between accuracy and simplicity.

- **Soft-delete entity patterns:** Using visibility flags instead of structural mutation during iteration.

- **Continuous collision detection:** Why tunneling occurs at high velocity and how sweep-based collision fixes it.

- **Architectural self-review:** Identifying flaws in working systems and proposing production-grade alternatives.

---

## Future Scope

| Improvement | Technical Detail |
|---|---|
| Dedicated game loop thread | `ScheduledExecutorService` for updates + `SwingUtilities.invokeLater()` for rendering |
| Delta-time physics | Multiply velocity by elapsed frame time |
| Sweep-based collision | Ray-cast along velocity vector to prevent tunneling |
| ECS refactor | Decouple entities from behavior using Entity-Component-System architecture |
| Lives system | Add respawn logic and remaining-life tracking |
| Multiple levels | Load map configurations from external data |
| Maven build | Reproducible builds and fat-JAR packaging |
| Unit tests (`JUnit 5`) | Isolated testing for collision and game state logic |
| JavaFX port | Hardware-accelerated rendering and animation timeline support |

---

## Repository Philosophy

This repository is built on the principle that **understanding the flaws in your own work is a more valuable engineering signal than hiding them.** The code is functional, the architecture is honest about its limitations, and the documentation is written the way a senior engineer would review it — not the way a student would defend it.

The goal is not to present a perfect system. It is to present a system whose tradeoffs can be articulated precisely.

---

## License

MIT License. See LICENSE for details.

---

## Contact

### Connect With Me

[![GitHub](https://img.shields.io/badge/GitHub-Heramb1221-black?style=for-the-badge&logo=github)](https://github.com/Heramb1221)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Heramb%20Chaudhari-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/heramb-chaudhari)

[![Email](https://img.shields.io/badge/Email-hchaudhari1221%40gmail.com-red?style=for-the-badge&logo=gmail)](mailto:hchaudhari1221@gmail.com)

---
