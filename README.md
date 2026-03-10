# LEGO SPIKE Prime Line-Following Robot

## Overview

`myfile` is a **MicroPython** program written for the **LEGO SPIKE Prime** robotics platform. It implements an autonomous line-following robot capable of navigating a course with coloured markings, handling turns, mode switches, and a rescue-zone search.

---

## Hardware Setup

| Component | Port |
|---|---|
| Drive motors (wheel pair) | A (left) & B (right) |
| Distance sensor | D |
| Left colour sensor | E |
| Right colour sensor | F |
| Motion / IMU sensor | built-in (TOP face) |

---

## Key Features

### Line Following
The robot reads two colour sensors (left = port E, right = port F) and reacts to what they detect:

| Left sensor | Right sensor | Action |
|---|---|---|
| Space (off-line) | Space (off-line) | Go straight (`go_straight`) |
| Line | Space | Steer left (`steer_car(-80, …)`) |
| Space | Line | Steer right (`steer_car(80, …)`) |
| Line | Line | Stop (intersection) |
| Yellow (either) | — | Stop |
| Green (both) | — | Stop (rescue zone reached) |
| Green (left only) | — | Turn CCW 90 °, begin rescue search |
| Green (right only) | — | Turn CW 90 °, begin rescue search |

### Speed Control
- **`base_speed`** starts at 180 (large motor) or 200 (medium motor).
- Accelerates up to **`MAX_SPEED`** (280) on long straights.
- Decelerates after sharp turns, tracked by **`turn_cnt`**.
- Adjusts for ramps using the IMU pitch angle.

### Two-Colour Modes
`switch_mode()` toggles between:
- **Mode 1 – Normal** : black line on white background (😊).
- **Mode 2 – Dark / Inverted** : white line on black background (👻).

The robot automatically switches when it detects an inverted surface mid-run.

### Turning
- **`steer_car(ratio, speed)`** – steers the car back onto the line using a continuous motor-ratio approach; uses the IMU heading to detect over-rotation and self-correct.
- **`turn_any_degree(direction, degree)`** – performs a pivot turn of the given angle (CCW = direction 1, CW = direction 2) guided by IMU yaw. The main loop passes 50 ° for green-zone turns.
- **`go_circle(direction, degree)`** – drives an arc (CW = direction 1, CCW = direction 2) until the far colour sensor finds the line again or the target angle is reached.

### Rescue Zone Search
`find_rescue(search_degree, interval)` sweeps the distance sensor left and right, records distance readings at each 10 ° step, then pivots to face the closest detected object (the rescue target).

---

## Program Flow

```
Wait for RIGHT button press
    (press LEFT to preview circle motion)
↓
Reset IMU yaw
Start in Mode 2 (dark), immediately switch to Mode 1 (normal)
↓
Main loop (while run == 1)
    Read both colour sensors → dispatch to the appropriate handler
↓
Stop motors when run ≠ 1
```

---

## Global State

| Variable | Description |
|---|---|
| `run` | `1` = running, `0` = paused, `-1` = error / lost line |
| `mode` | `1` = normal (black line), `2` = dark (white line) |
| `base_speed` | Current cruising speed (MIN_SPEED … MAX_SPEED) |
| `turn_cnt` | Accumulated turn amount; used to slow down after curves |
| `straight_cnt` | Consecutive straight-line ticks; used to trigger acceleration |
| `tick` | Global time counter (× 10 ms per tick) |
| `intersection` | Intersection counter (reserved for future logic) |
| `dark_side` | Flag for inverted-colour half of the course |

---

## Known Issue

Line 211 contains a comparison (`==`) where an assignment (`=`) was likely intended:

```python
trial == 2   # ← should be  trial = 2
```

This means the second search direction is never triggered and `trial` stays at `1` indefinitely.
