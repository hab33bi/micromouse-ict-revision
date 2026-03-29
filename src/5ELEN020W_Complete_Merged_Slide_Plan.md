# 5ELEN020W MICRO-MOUSE: COMPLETE REVISION SLIDE DECK PLAN
## Final Merged Version — Exhaustive Exam Revision

---

# HOW TO USE THIS DOCUMENT

This is a complete, slide-by-slide plan for building exam revision slides for module **5ELEN020W** at the University of Westminster. Every slide includes: a title, full bullet-point content, code/formulas where applicable, and a detailed visual/diagram description. Build each slide directly from this specification.

**Total slides: 62** (organised into 14 sections)

---

# STRUCTURAL NOTES & JUSTIFICATIONS

The following structural changes were made when merging the two source documents:

1. **New Section 1 (Project Overview & Stages)** added — the master prompt covers the three-stage project breakdown and top-level solution flow, which the pre-planned slides omitted entirely. This is essential context that should open the deck.

2. **New Section 2 (Documenting Behaviour & Logic)** added — the master prompt extensively covers behavioural charts vs flowcharts and why state diagrams suit the micro-mouse. This is examinable methodology content missing from the pre-planned slides.

3. **New Section 3 (State Machine Model)** added — the master prompt contains detailed state diagrams, transition tables, and sub-states. This is the architectural backbone of the entire micro-mouse software and was absent from the pre-planned slides.

4. **New Section 4 (Software Architecture)** added — the main-loop + ISR parallel execution model, with code, was only in the master prompt. It is fundamental to understanding how the robot operates.

5. **Section on State Variable & Pattern Matching** expanded into its own section — the pre-planned slides had a single slide on bitwise AND. The master prompt has full code examples for update_state(), pattern checking functions, and optimised nested-IF approaches. These deserve multiple slides.

6. **Sensor Noise section** (pre-planned Section 5) merged with **Sensor Confidence Algorithms** (pre-planned Section 6) and **master prompt's confidence algorithms** into a single expanded section — they are the same conceptual domain (making sensor reads reliable). The majority vote walkthrough from the master prompt is added as dedicated slides.

7. **Timing Mechanism** elevated to its own section — the master prompt contains extensive content on tick counting, motor speed formulas, sensor sampling formulas, choosing 'm', and worked examples. The pre-planned slides only had one Ticker slide buried in MBED programming. This content deserves dedicated treatment.

8. **MBED Programming section** restructured — Timer class, Ticker class, InterruptIn, and Ping/Echo code are grouped together with the full code examples from the master prompt.

9. **Pre-planned hardware sections preserved and enhanced** — L297/DS2003, kinematics, LM7805, batteries all kept from pre-planned slides with additional detail layered in.

10. **Four new closing slides added** — Complete Task Breakdown, System Block Diagram, Exam Tips, and Practice Questions, all from the master prompt's recommendations.

---

# SECTION 1: PROJECT OVERVIEW & STAGES
*Justification: Essential context. Opens the deck with what the project is and how it is structured.*

---

## SLIDE 1: The Micro-Mouse Problem — What Is It?

**Content:**
- The IEEE Micromouse is an autonomous robot that navigates and solves a maze without human intervention.
- The maze is an 8×8 grid of cells, each cell a square with possible walls on any of its four sides.
- The robot must: explore the maze, map it, find the target cell (usually the centre), then perform a speed run along the optimal path.
- The project is studied as part of module 5ELEN020W at the University of Westminster, combining software engineering and electronics.

**Visual:** Photograph or illustration of a typical micromouse robot on an 8×8 maze. If unavailable, a top-down diagram of an 8×8 grid maze with a small robot icon at the start (corner) and a target icon at the centre.

---

## SLIDE 2: Three Program Stages

**Content:**
- The software solution is built incrementally in three stages, each building on the previous:
  - **Stage 1: Tunnel Run** — The robot drives forward through a straight corridor. Tests basic motor control and sensor reading. No turning logic required.
  - **Stage 2: Hand-on-Wall Algorithm** — The robot follows the left (or right) wall to navigate through the maze. Tests turning logic, state transitions, and sensor-based decision making. Does NOT map the maze.
  - **Stage 3: Maze Mapping Algorithm** — The robot maps the entire maze using an algorithm (e.g. Lee's Algorithm), stores wall positions, then performs a speed run from start to target via the shortest path. Full solution.
- **Common elements across ALL stages** (code reuse):
  - Motor drivers (stepping, direction, speed control)
  - Sensor drivers (read, filter, interpret)
  - State definitions (Travelling Forward, Turning, etc.)
- Building incrementally reduces debugging complexity — get Stage 1 working perfectly before adding Stage 2 logic.

**Visual:** Three stacked blocks arranged vertically (or horizontally), labelled "Stage 1: Tunnel Run", "Stage 2: Hand-on-Wall", "Stage 3: Maze Mapping". Arrows show progression upward. To the right, a sidebar lists "Shared Components: Motor Drivers, Sensor Drivers, State Definitions" with dotted lines connecting to all three stages.

---

## SLIDE 3: Top-Level Solution — Three Phases

**Content:**
- The full Stage 3 solution has three sequential runtime phases:
  1. **Initialise** — Set up microcontroller hardware (timers, GPIO pins, interrupts). Initialise software variables (state = IDLE, position = (0,0), orientation = NORTH, clear wall map array, clear Lee's map array).
  2. **Map Maze** — Explore the entire maze (or enough of it) using Left-Hand-on-Wall, Lee's Algorithm, or another exploration strategy. At each cell, record which walls are present. Build the wall map and Lee's number map.
  3. **Speed Run** — Navigate from the starting square to the target square using the shortest path found during mapping. Follow descending Lee's numbers. Can use higher motor speed since the path is known and no exploration is needed.
- After the speed run, the maze run is complete.

**Visual:** Three rectangular boxes in a horizontal row connected by right-pointing arrows:
- Box 1: "Initialise" (light grey fill)
- Box 2: "Map Maze" (blue fill, bold border) — annotation below in smaller text: "Left-Hand Wall / Lee's Algorithm"
- Box 3: "Speed Run" (green fill) — annotation above in smaller text: "Follow optimal path to target"
Arrow labels: "Hardware & variables ready" between boxes 1→2, "Maze fully mapped" between boxes 2→3.

---

# SECTION 2: DOCUMENTING BEHAVIOUR & LOGIC
*Justification: The lecture explicitly covers documentation methods (behavioural charts, flowcharts, state diagrams) as a key topic. Examinable.*

---

## SLIDE 4: Ways to Document Behaviour and Logic

**Content:**
- Before writing code, the micro-mouse's behaviour must be documented using formal diagrams.
- Two categories of documentation:
  - **Charts:**
    - Behavioural Charts — describe WHAT the system should do at a functional level.
    - Flowcharts — describe HOW the system implements the required behaviour at a code/logic level.
  - **State Diagrams** — describe the system as a set of states with transitions triggered by events/conditions.
- Both charts and state diagrams describe the **sequence** in which things happen, but at different levels of abstraction.
- Because the micro-mouse is fundamentally a state machine, **state diagrams** are the most natural and recommended way to describe its behaviour.

**Visual:** A simple hierarchy diagram. Top node: "Documentation Methods". Two child nodes: "Charts" (with sub-nodes "Behavioural" and "Flow") and "State Diagrams". The "State Diagrams" node is highlighted/starred with annotation: "Best fit for the micro-mouse".

---

## SLIDE 5: Behavioural Chart vs Flowchart — Critical Distinction

**Content:**
- **Behavioural Chart:**
  - Describes WHAT to do at a high, functional level.
  - Uses natural-language-style decisions: "Is there a wall on the left?" → "Turn left".
  - Does NOT specify implementation details (no variables, no register checks).
  - Audience: anyone understanding the problem domain.
- **Flowchart:**
  - Describes HOW to implement the behaviour in software.
  - Uses code-level logic: "State == Turning?" → "State = Turn_Left" → "L1, L2, L3 == 0?".
  - Specifies variable names, conditions, and program flow.
  - Audience: the programmer implementing the solution.
- **Key exam point:** Both describe sequence, but at different abstraction levels. Do not confuse them. An exam question may ask you to draw one or the other — know the difference.

**Visual:** Side-by-side comparison. Left panel labelled "Behavioural Chart (WHAT)": simple flowchart with decision diamond "Is there a wall on the left?" → "Yes: Continue forward" / "No: Turn left". Right panel labelled "Software Flowchart (HOW)": detailed flowchart with decision diamond "State == Turning?" → "N: State = Turn_Left" → "L1, L2, L3 == 0?" → "Y: Execute turn sequence". A dividing line between them with label "Same behaviour, different abstraction level".

---

# SECTION 3: THE MICRO-MOUSE AS A STATE MACHINE
*Justification: The state machine model is the central architectural concept. The lecture devotes multiple slides to states, sub-states, and state diagrams. This section recreates all of them.*

---

## SLIDE 6: Why a State Machine?

**Content:**
- The micro-mouse operates as a **finite state machine (FSM)**:
  - At a **fixed, known time period**, sensors are read.
  - Based on sensor values, the **next STATE** is determined.
  - The current STATE controls **what actions** are performed (e.g. step motors forward, turn left, stop).
- The time interval between sensor reads and actions is controlled by either:
  - A **wait subroutine** (simple but blocking — less precise).
  - A **clock tick signal** (interrupt-driven — precise and recommended).
- Because the micro-mouse is a state machine, its behaviour is best described and documented using **state diagrams**.
- All decision-making reduces to: **Read sensors → Determine state → Execute actions for that state → Repeat.**

**Visual:** Circular diagram showing the FSM loop: "Read Sensors" → "Determine Next State" → "Execute Actions" → (arrow back to "Read Sensors"). In the centre: "STATE variable". Annotation: "Repeats every tick period".

---

## SLIDE 7: States — Maze-Solving Task (High Level)

**Content:**
- At the highest level, the maze-solving task has these states:
  - **Mapping** — Robot is exploring the maze, recording walls, building the map.
  - **Back to Start Run** — Robot returns to the starting square after mapping is complete (or partially complete).
  - **Speed Run** — Robot navigates from start to target via the optimal (shortest) path.
  - **Maze Run Complete** — Robot has reached the target. Task finished.
- These are **task-level states** — they govern which overall behaviour the robot is performing.
- Within each of these, there are **navigation-level states** (see next slides).

**Visual:** Linear state diagram with four rounded rectangles in a row: "Mapping" → "Back to Start" → "Speed Run" → "Complete". Transition labels: "Maze fully explored", "Reached start", "Reached target".

---

## SLIDE 8: States — Lee's Algorithm Sub-States

**Content:**
- When the robot is in the "Mapping" task state and using Lee's Algorithm, it cycles through these sub-states:
  - **Start Square** — Robot is at the starting cell. Lee's numbers initialised.
  - **Follow Descending Numbers** — Robot moves toward the target by following cells with decreasing Lee's values.
  - **Re-map** — A new wall is discovered that invalidates the current path. Lee's numbers must be recalculated from the current position.
  - **Target Found** — Robot has reached the target cell. Mapping phase can end (or continue for completeness).
- Re-mapping is critical: the robot's initial Lee's map is based on incomplete information. Every time a new wall is found, the flood-fill must be re-run.

**Visual:** State diagram with four states: "Start Square" → "Follow Descending Numbers" → "Target Found". A return arrow from "Follow Descending Numbers" to "Re-map" labelled "New wall found". Arrow from "Re-map" back to "Follow Descending Numbers" labelled "Numbers recalculated".

---

## SLIDE 9: States — Navigation Task (Movement)

**Content:**
- Regardless of which task-level state the robot is in, it is always in one of these navigation states:
  - **Travelling Forward** (superstate — contains sub-states):
    - **Not Turning** — Moving straight ahead, possibly with steering correction.
    - **About to Turn** — A gap (missing side wall) has been detected. Robot continues to the cell centre before turning.
    - **About to Stop** — A front wall has been detected. Robot continues to the cell centre before stopping.
    - **STEER CORRECTION** — Within "Not Turning", adjusting trajectory to stay centred.
    - **ACCELERATING** — Ramping up motor speed.
  - **Turning** — Executing a turn:
    - TURN LEFT
    - TURN RIGHT
  - **U-Turn** — Executing a 180° turn (dead end).
  - **Stopped** — Robot has halted (e.g. at a dead end before U-turning).

**Visual:** Hierarchical list or tree diagram showing "Navigation States" as root, with "Travelling Forward" as a branch containing its sub-states, "Turning" with TURN LEFT / TURN RIGHT, "U-Turn", and "Stopped" as siblings.

---

## SLIDE 10: Top-Level Navigation State Diagram (CRITICAL — Must Know for Exam)

**Content:**
- This diagram shows how the robot transitions between navigation states:
- **States:**
  1. Initialise
  2. Travelling Forward (superstate) containing:
     - Not Turning
     - About to Turn
     - About to Stop
  3. Turning
  4. Stopped
- **Transitions:**
  - `Initialise` → `Not Turning` : "Initialisation complete"
  - `Not Turning` → `About to Turn` : "Side wall gone" (gap detected — possible turn)
  - `About to Turn` → `Turning` : "In square centre" (robot has reached cell midpoint, safe to turn)
  - `Turning` → `Not Turning` : "Turn complete" (turn finished, resume forward travel)
  - `Not Turning` → `About to Stop` : "Front wall present" (dead end or wall ahead)
  - `About to Stop` → `Stopped` : "In square centre" (robot stops at cell centre)
  - `Stopped` → U-Turn or other action
- **Key concept:** The robot must reach the **centre of a square** before executing a turn or stop. This ensures consistent positioning for the next action.

**Visual (MUST RECREATE — this is the main diagram from Slide 9 of the original lecture):**
- Top-left: Small rectangle "Initialise" with arrow downward labelled "Initialisation complete".
- Large shaded rectangle labelled "Travelling Forward" containing three internal state boxes:
  - Top: "Not Turning"
  - Middle: "About to Turn"
  - Bottom (with rounded/dashed border): "About to Stop"
- Internal transitions: "Not Turning" → "About to Turn" (label: "Side wall gone"). "Not Turning" → "About to Stop" (label: "Front wall present").
- Outside the superstate, right side: Rectangle "Turning" — arrow from "About to Turn" → "Turning" (label: "In square centre"), arrow from "Turning" → "Not Turning" (label: "Turn complete").
- Below "Turning": Ellipse "Stopped" — arrow from "About to Stop" → "Stopped" (label: "In square centre"). Annotation near "Stopped": "Perhaps you would have a U-turn here."
- A legend on the right listing all states: Initialise, Travelling Forward, Not Turning, About to Turn, About to Stop, Stopped, Turning.

---

## SLIDE 11: Steering Correction Sub-States

**Content:**
- Within the "Not Turning" state, the robot continuously corrects its trajectory to stay centred in the corridor.
- **Sub-states:**
  - **No Steering Correction** — Robot is centred between walls. Both motors step at equal speed.
  - **Steer Left** — Robot has drifted too far to the right. Left motor slows (or right motor speeds up) to correct.
  - **Steer Right** — Robot has drifted too far to the left. Right motor slows (or left motor speeds up) to correct.
- **Transitions:**
  - `No Steering Correction` → `Steer Left` : "Too far right" (right-side sensors detect closer wall)
  - `No Steering Correction` → `Steer Right` : "Too far left" (left-side sensors detect closer wall)
  - `Steer Left` → `No Steering Correction` : "Correction complete"
  - `Steer Right` → `No Steering Correction` : "Correction complete"
- Steering correction is implemented by stepping one motor faster than the other (differential drive). See Timing section for details.

**Visual:** Three-state diagram in triangular arrangement. Top: rounded rectangle "No Steering Correction". Bottom-left: "Steer Left". Bottom-right: "Steer Right". Arrows: Top→Bottom-left ("Too far right"), Top→Bottom-right ("Too far left"), Bottom-left→Top ("Correction complete"), Bottom-right→Top ("Correction complete"). All enclosed in a dashed superstate box labelled "Not Turning".

---

# SECTION 4: SOFTWARE ARCHITECTURE
*Justification: The lecture provides a specific main-loop + ISR architecture with code. This is the implementation backbone.*

---

## SLIDE 12: Program Structure — Main Loop + Interrupt Handler

**Content:**
- The micro-mouse software has two concurrent execution paths:
  1. **Main Loop** (runs continuously):
     - Reads sensor pattern.
     - Calls `update_state()` to determine the next state based on the sensor pattern.
     - May also handle high-level logic (mapping updates, speed run decisions).
  2. **Interrupt Handler / ISR** (runs on every timer tick):
     - Increments `tick_count`.
     - Every `n` ticks, executes the motor action for the current state via a `switch(state)` statement.
     - If the robot is NOT turning AND has reached the middle of a square: calls `Update_Position()`.
     - If a turn has just completed: calls `Update_Orientation()`.
- The **state variable** is the shared link between the main loop and the ISR.
- This architecture separates **decision-making** (main loop) from **real-time motor control** (ISR).

**Visual:** Split diagram. Left column: "Main Loop" flowchart — Start → Read Sensors → update_state() → (loop back). Right column: "Interrupt Handler (Tick)" flowchart — tick_count++ → if tick_count==n → switch(state): case actions → if !Turning && Middle_of_Square → Update_Position() → if Turning_Complete → Update_Orientation(). A dashed line between the two columns with a shared box labelled "state" (the shared variable). Arrows from main loop writing to "state" and ISR reading from "state".

---

## SLIDE 13: ISR Pseudo-Code

**Content:**
- The interrupt handler is called on every timer tick. Its pseudo-code:

```c
// Interrupt Handler — called on every Ticker tick
void tick_handler() {
    tick_count++;
    
    // Motor actions — execute every n ticks
    if (tick_count == n) {
        switch (state) {
            case FORWARD:     step_both_motors();    break;
            case TURN_LEFT:   step_left_turn();      break;
            case TURN_RIGHT:  step_right_turn();     break;
            case STEER_LEFT:  step_steer_left();     break;
            case STEER_RIGHT: step_steer_right();    break;
            case STOPPED:     /* do nothing */       break;
        }
        tick_count = 0;  // reset counter
    }
    
    // Position update — only when going straight and at cell centre
    if (state != TURNING && middle_of_square()) {
        Update_Position();  // increment X or Y based on orientation
    }
    
    // Orientation update — after a turn completes
    if (state == TURNING_COMPLETE) {
        Update_Orientation();  // e.g. NORTH + LEFT_TURN = WEST
    }
}
```

- **Critical rule:** The ISR execution time must be **shorter** than the Ticker period. If the ISR takes longer to run than the interval between ticks, ticks will be missed and timing will break.

**Visual:** The code above displayed in a syntax-highlighted code block. Annotations with arrows pointing to key lines: "Motor speed control" pointing to the switch block, "Dead reckoning" pointing to Update_Position(), "Keep ISR fast!" as a red warning box.

---

# SECTION 5: SENSOR LAYOUT, PATTERN MATCHING & STATE ENCODING
*Justification: Merges the master prompt's detailed sensor mapping and pattern-matching code with the pre-planned bitwise operations slide. This is a multi-slide topic.*

---

## SLIDE 14: Sensor Layout and Bit Mapping

**Content:**
- The robot has **7 sensors** — 3 on the left side, 1 at the front, 3 on the right side.
- All 7 sensor values are packed into a single 8-bit `char` variable for efficient processing:

```
Bit position:  7     6     5     4     3     2     1     0
Sensor:       [x]   [LR]  [LC]  [LL]  [F]   [RL]  [RC]  [RR]
              MSB   Left   Left  Left  Front Right  Right Right
             (unused) Far  Centre Near        Near  Centre Far
```

- **LR** = Left Far (bit 6), **LC** = Left Centre (bit 5), **LL** = Left Near (bit 4)
- **F** = Front (bit 3)
- **RL** = Right Near (bit 2), **RC** = Right Centre (bit 1), **RR** = Right Far (bit 0)
- **Convention:** `1` = wall detected, `0` = no wall (open space).
- Bit 7 (MSB) is unused (always 0 or don't-care).

**Visual:** Two-part diagram:
- **Top:** Top-down view of the robot (rectangle shape, front facing upward). Seven sensor positions marked with labelled dots/arrows: three along the left edge (LR far back, LC middle, LL near front), one at the front centre (F), three along the right edge (RL near front, RC middle, RR far back).
- **Bottom:** 8-box register diagram showing the bit layout: `| x | LR | LC | LL | F | RL | RC | RR |` with bit numbers 7→0 below and "MSB" / "LSB" labels at the ends.

---

## SLIDE 15: Basic State Update Function — Pattern Matching

**Content:**
- The `update_state()` function determines the next state by checking the sensor pattern against known pattern tables for each possible state:

```c
char update_state(char sensor_pattern, char old_state) {
    char match = 0;
    
    match = check_TURN_LEFT_patterns(sensor_pattern);
    if (match == 1) { return TURN_LEFT; }
    
    match = check_TURN_RIGHT_patterns(sensor_pattern);
    if (match == 1) { return TURN_RIGHT; }
    
    match = check_U_TURN_patterns(sensor_pattern);
    if (match == 1) { return U_TURN; }
    
    match = check_STEER_LEFT_patterns(sensor_pattern);
    if (match == 1) { return STEER_LEFT; }
    
    match = check_STEER_RIGHT_patterns(sensor_pattern);
    if (match == 1) { return STEER_RIGHT; }
    
    return old_state;  // no match — keep current state
}
```

- The function cascades through checks in **priority order**: Turn Left → Turn Right → U-Turn → Steer Left → Steer Right → no change.
- Each `check_X_patterns()` function contains a switch statement listing all known sensor patterns for that state.

**Visual:** Flowchart showing the cascade: Sensor pattern enters at top → diamond "Match TURN_LEFT?" → Y: return TURN_LEFT / N: → diamond "Match TURN_RIGHT?" → Y: return TURN_RIGHT / N: → diamond "Match U_TURN?" → ... → final box "return old_state".

---

## SLIDE 16: Pattern Checking Function (Per State)

**Content:**
- Each state has its own pattern-checking function. Example for TURN_LEFT:

```c
char check_TURN_LEFT_patterns(char pattern) {
    char match;
    switch (pattern) {
        case 0b00001111:  match = 1; break;  // example pattern
        case 0b00001110:  match = 1; break;  // another valid pattern
        case 0b00001101:  match = 1; break;  // ...
        case 0b00001100:  match = 1; break;
        // ... all valid TURN_LEFT sensor patterns listed ...
        default:          match = 0; break;  // not a TURN_LEFT pattern
    }
    return match;
}
```

- Each `case` is a specific 7-bit sensor pattern that indicates this state.
- The patterns are determined empirically — test the robot and record which sensor patterns correspond to which navigation situations.
- The `default` case returns 0 (no match).

**Visual:** Table showing example sensor patterns for TURN_LEFT. Columns: Pattern (binary), LR, LC, LL, F, RL, RC, RR, Meaning. Example row: `0000111` → 0,0,0,0,1,1,1 → "No left wall, front and right walls present — turn left".

---

## SLIDE 17: The Problem — Slow Sequential Checking

**Content:**
- The basic `update_state()` function checks **every** pattern table sequentially. This is slow.
- Example: if the sensor pattern actually represents a U-Turn, the function must first check:
  1. All Turn Left patterns (no match)
  2. All Turn Right patterns (no match)
  3. All Steer Left patterns (no match)
  4. All Steer Right patterns (no match)
  5. Finally, U-Turn patterns (match!)
- This wastes valuable CPU time, especially in a real-time system where the ISR must complete before the next tick.
- **Solution:** Use **bitwise AND masking** to quickly categorise the sensor pattern before doing detailed checks.

**Visual:** Timeline/bar diagram showing CPU time. A long bar divided into segments: "Check Turn Left (no match)", "Check Turn Right (no match)", ..., "Check U-Turn (MATCH!)". The majority of the bar is wasted time. Below, a shorter bar showing the optimised approach: "Mask check (instant)" → "Check U-Turn (MATCH!)".

---

## SLIDE 18: Bitwise AND Masking for Fast Pattern Categorisation

**Content:**
- **Key insight:** All patterns belonging to the same state share **common bits**. Use bitwise AND with a mask to instantly check if those common bits are set.
- **Bitwise AND (`&`) operator:** Compares two binary numbers bit-by-bit. Result bit = 1 only if BOTH input bits = 1.
  - Example: `00000001 & 00000011 = 00000001` (i.e. `1 & 3 = 1`)
- **Application — checking for left turn possibility:**
  - All TURN_LEFT patterns have one thing in common: the left sensors (LL, LC, LR — bits 4, 5, 6) are all `0` (no left wall).
  - Mask: `0x70` = `01110000` in binary. This isolates bits 6, 5, 4.
  - Test: `if ((sensor_pattern & 0x70) == 0x00)` → left sensors all clear → worth checking TURN_LEFT patterns.
- **Application — checking for right turn possibility:**
  - Mask: `0x07` = `00000111`. Isolates bits 2, 1, 0 (right sensors).
  - Test: `if ((sensor_pattern & 0x07) == 0x00)` → right sensors all clear → check TURN_RIGHT patterns.
- **Application — checking for deviation right:**
  - Mask: `0x44` = `01000100`. Checks specific bits.
  - Test: `if ((sensor_pattern & 0x44) != 0x00)` → deviation-right indicators present → check deviation patterns.
- **Other useful bitwise operators:**
  - `|` (OR) — combine bit patterns.
  - `^` (XOR) — flip/toggle specific bits.

**Visual:** Three-panel binary operation diagram:
- Panel 1: Sensor byte `0b0000_1100` ANDed with mask `0x70` = `0b0111_0000`. Show the operation bit by bit. Result = `0b0000_0000`. Annotation: "Left sensors all clear — possible left turn."
- Panel 2: Sensor byte `0b0110_1000` ANDed with mask `0x07` = `0b0000_0111`. Result = `0b0000_0000`. Annotation: "Right sensors all clear — possible right turn."
- Panel 3: Sensor byte `0b1100_0100` ANDed with mask `0x44` = `0b0100_0100`. Result = `0b0100_0100` (non-zero). Annotation: "Deviation-right bits set — check deviation patterns."

---

## SLIDE 19: Optimised State Update Function with Nested Checks

**Content:**
- The optimised `update_state()` uses bitwise AND masks as **pre-filters** before checking detailed pattern tables:

```c
char update_state(char sensor_pattern, char old_state) {
    char match = 0;
    
    // Pre-filter: only check TURN_LEFT if left sensors show no wall
    if ((sensor_pattern & 0x70) == 0x00) {
        match = check_TURN_LEFT_patterns(sensor_pattern);
        if (match == 1) { return TURN_LEFT; }
    }
    
    // Pre-filter: only check TURN_RIGHT if right sensors show no wall
    if ((sensor_pattern & 0x07) == 0x00) {
        match = check_TURN_RIGHT_patterns(sensor_pattern);
        if (match == 1) { return TURN_RIGHT; }
    }
    
    // Pre-filter for deviation checks
    if ((sensor_pattern & 0x44) != 0x00) {
        match = check_DEVIATE_RIGHT_patterns(sensor_pattern);
        if (match == 1) { return STEER_RIGHT; }
    }
    
    // ... more pre-filtered checks ...
    
    return old_state;
}
```

- **Why this is faster:** The AND mask check is a single CPU instruction. If the mask test fails, the entire pattern table for that state is skipped. In the worst case, performance is similar to the basic version. In the average case, it is significantly faster.

**Visual:** Flowchart showing the optimised cascade: sensor_pattern enters → diamond "& 0x70 == 0?" → Y: check TURN_LEFT table → N: skip → diamond "& 0x07 == 0?" → Y: check TURN_RIGHT table → N: skip → ... Shows how branches are pruned.

---

# SECTION 6: MAZE MAPPING & LEE'S ALGORITHM
*Justification: Kept from pre-planned slides, enhanced with master prompt's additional detail on Lee's algorithm states.*

---

## SLIDE 20: Introduction to Lee's Algorithm

**Content:**
- Lee's Algorithm is an **exhaustive breadth-first search (BFS)** guaranteed to find the shortest path in the maze — if one exists.
- It works in two phases:
  1. **Flood fill (numbering):** Assign distance values to every reachable cell, expanding outward from the target.
  2. **Trace back (navigation):** Follow descending numbers from the current position to the target.
- The algorithm is computationally simple and well-suited to the limited resources of a microcontroller.
- It must be **re-run** every time a new wall is discovered that invalidates the current path.
- Lee's Algorithm states (see State Machine section): Start Square → Follow Descending Numbers → (Re-map if new wall found) → Target Found.

**Visual:** High-level flowchart: "Set target cell = 0" → "Flood fill outward (+1 per cell)" → "Robot follows descending numbers" → diamond "New wall found?" → Y: "Re-run flood fill" (loop back) / N: "Continue to target".

---

## SLIDE 21: Lee's Algorithm — Numbering the Grid (Flood Fill)

**Content:**
- **Step-by-step process:**
  1. Set the target cell value to `0`.
  2. Find all unblocked adjacent cells (no wall between them and the current cell). Set them to `1`.
  3. Find all unblocked cells adjacent to cells labelled `1` that haven't been numbered yet. Set them to `2`.
  4. Continue expanding: each new wavefront gets the previous value + 1.
  5. Stop when all reachable cells are numbered (or when the start cell is reached).
- Walls block propagation — a cell behind a wall is NOT adjacent even if it is physically next to the current cell.
- The number in each cell represents the **minimum number of steps** from that cell to the target.
- In an 8×8 maze, the maximum possible Lee's value is **63** (worst case: snake path through all 64 cells). This fits in 6 bits.

**Visual:** An 8×8 grid (or simplified 5×5 for clarity) showing the flood fill in progress. Target cell (centre) = 0. Adjacent cells = 1. Next ring = 2. Show walls blocking propagation in certain directions. Use colour gradient: low numbers = green, high numbers = red.

---

## SLIDE 22: Lee's Algorithm — Navigation & Recalculation

**Content:**
- **Navigation (Trace-back):**
  - From any cell, the robot moves to the adjacent cell with the **lowest Lee's number**.
  - This naturally traces the shortest path to the target.
  - At each step, the robot decrements by at least 1.
- **Recalculation (Re-map):**
  - During exploration, the robot discovers walls that weren't in its initial (empty) map.
  - When a new wall is found, the current Lee's numbers may be invalid — a previously open path is now blocked.
  - The robot must **re-run the flood fill** from its current position (or from the target) incorporating the new wall data.
  - This may happen multiple times during a single maze exploration.
- **Speed Run:** After the maze is fully mapped, the final Lee's numbers represent the true shortest path. The speed run follows these numbers without needing recalculation.

**Visual:** Two side-by-side 5×5 grids:
- Grid 1 (Before): Lee's numbers filled in, with a path traced in green from start to target.
- Grid 2 (After): A new wall discovered (shown in red). Lee's numbers recalculated — some numbers changed. New path traced in green, showing it routes around the new wall.

---

# SECTION 7: DATA STORAGE & MEMORY MANAGEMENT
*Justification: Expanded from pre-planned slides 4-5. Added position/orientation tracking from master prompt.*

---

## SLIDE 23: Data Variables Required

**Content:**
- The micro-mouse must store the following data in memory:
  - **Position:** `X_POSITION` (char/int), `Y_POSITION` (char/int) — current cell coordinates in the 8×8 grid (0–7 each).
  - **Orientation:** `ORIENTATION` (char/enum) — current heading: NORTH, SOUTH, EAST, or WEST. Determines which direction is "forward".
  - **Wall Map:** `wall_map[8][8]` (char array) — stores which walls are present around each cell.
  - **Lee's Map:** `lees_map[8][8]` (char array) — stores the Lee's algorithm flood-fill values for each cell.
  - **State:** `state` (char/enum) — current navigation state (FORWARD, TURNING, STOPPED, etc.).
  - **Sensor Pattern:** `sensor_pattern` (char) — current sensor reading packed into 8 bits.

**Visual:** Table with columns: Variable Name, Data Type, Size, Range, Description. Rows for each variable listed above.

---

## SLIDE 24: Wall Map — 4-Bit Encoding Per Cell

**Content:**
- Each cell in the 8×8 maze can have walls on up to 4 sides: North, South, East, West.
- Encode wall presence as 4 bits per cell:
  - Bit 3 = North wall (1 = wall present, 0 = open)
  - Bit 2 = South wall
  - Bit 1 = East wall
  - Bit 0 = West wall
- Example: `0b1010` = walls on North and East, open on South and West.
- This means each cell needs only **4 bits** (half a byte). Two cells can be packed into a single byte.
- Total memory for 8×8 maze: 64 cells × 4 bits = 256 bits = **32 bytes** (if packed) or 64 bytes (if using a full char per cell for simplicity).
- **Important:** When the robot records a wall on one side of a cell, it must also record the corresponding wall on the adjacent cell. E.g., a North wall on cell (3,4) means a South wall on cell (3,5).

**Visual:** Single maze cell diagram — a square with four sides. Each side labelled with its bit: N (bit 3), S (bit 2), E (bit 1), W (bit 0). Example: walls drawn on North and East sides, with the 4-bit value `1010` shown inside the cell. Below, an 8×8 grid with a few cells filled in showing their 4-bit wall codes.

---

## SLIDE 25: Lee's Map — 6-Bit Values

**Content:**
- The Lee's flood-fill value for each cell in an 8×8 maze ranges from 0 to a maximum of 63.
- 63 in binary = `0b00111111` — only 6 bits are needed.
- This fits in a `char` (8 bits) with 2 bits spare.
- The spare bits could be used for flags (e.g., "visited" flag, "on optimal path" flag).
- Total memory: 64 cells × 1 byte = **64 bytes**.

**Visual:** An 8-bit byte diagram: `| unused | unused | B5 | B4 | B3 | B2 | B1 | B0 |`. The 6 active bits highlighted in blue, the 2 unused bits greyed out. Annotation: "Max value = 63 = 0b00111111".

---

## SLIDE 26: Position and Orientation Tracking

**Content:**
- **Position updates:**
  - Position is updated when the robot crosses the **centre of a cell** while travelling forward (not turning).
  - Based on current orientation:
    - Facing NORTH: Y_POSITION += 1
    - Facing SOUTH: Y_POSITION -= 1
    - Facing EAST: X_POSITION += 1
    - Facing WEST: X_POSITION -= 1
- **Orientation updates:**
  - Orientation is updated when a **turn is completed**.
  - Turn Left: NORTH→WEST, WEST→SOUTH, SOUTH→EAST, EAST→NORTH
  - Turn Right: NORTH→EAST, EAST→SOUTH, SOUTH→WEST, WEST→NORTH
  - U-Turn: NORTH→SOUTH, SOUTH→NORTH, EAST→WEST, WEST→EAST
- **Landmark calibration:** Known features in the maze (e.g., start square walls, known dead ends) can be used to correct accumulated position errors.

**Visual:** 8×8 grid with robot icon at cell (2,3) facing NORTH (upward arrow). Show the position update for each direction with small arrows: if facing NORTH and crosses cell centre, new position = (2,4). Orientation change table: 4×3 table showing Current Orientation × Turn Direction = New Orientation.

---

# SECTION 8: SENSOR NOISE, FILTERING & CONFIDENCE
*Justification: Merges pre-planned Sections 5 (hardware noise) and 6 (algorithmic filtering) with the master prompt's detailed majority vote walkthrough and median/mean code. Unified because the topic is "making sensor reads reliable".*

---

## SLIDE 27: Ambient Light Interference — IS471F Solution

**Content:**
- **Problem:** IR proximity sensors can be blinded by ambient light (sunlight, fluorescent lights), causing false wall detections.
- **Solution:** The IS471F chip modulates the emitted IR beam at a specific frequency (e.g., 38 kHz).
- The receiver is tuned to only detect IR light at that modulation frequency, ignoring steady (DC) ambient light.
- This is the same principle used in TV remote controls.
- The IS471F outputs a digital signal: HIGH when a wall is detected (reflected modulated IR received), LOW when no wall.

**Visual:** Circuit schematic showing: IS471F chip → IR LED (emitting modulated beam) → Wall (reflecting beam) → IR Receiver (on IS471F) → Digital output to MCU pin. Annotate the modulated signal as a sine/square wave on the emitted beam, and show ambient light as a flat DC level that is filtered out.

---

## SLIDE 28: Mutual Sensor Interference

**Content:**
- **Problem:** If multiple IR sensors fire simultaneously, one sensor's emitted beam can be picked up by an adjacent sensor's receiver, causing false readings (crosstalk).
- **Solutions:**
  1. **Time Division Multiplexing (TDM):** Fire sensors sequentially, not simultaneously. Only one sensor emits at a time. This guarantees no crosstalk but requires careful timing.
  2. **Physical shielding:** Place opaque barriers between adjacent sensors to block stray reflections. Simpler but adds weight/bulk.
  3. **Combination:** Use both TDM and shielding for maximum reliability.
- TDM introduces a delay — all 7 sensors cannot be read instantaneously. The total read time = (number of sensors) × (time per sensor read).

**Visual:** Timing diagram showing 7 sensor channels on a vertical axis, time on horizontal axis. Each sensor fires in its own time slot (non-overlapping rectangular pulses). Label: "Time Division Multiplexing — one sensor active at a time."

---

## SLIDE 29: Mechanical Switch Bounce

**Content:**
- **Problem:** When a mechanical switch closes, the contacts physically bounce, causing rapid toggling between open and closed states for a few milliseconds before settling.
- This produces erratic voltage spikes — the MCU may interpret a single button press as multiple presses.
- Bounce duration: typically 1–20 ms depending on the switch.
- **Relevance to micro-mouse:** Any mechanical buttons used for start/mode selection, or bump sensors if used.

**Visual:** Voltage-time graph. X-axis: time (0 to ~20ms). Y-axis: voltage (0V to 5V). The graph shows a clean 5V→0V transition, then rapid bouncing (jagged up-and-down spikes) for ~10ms, then settling at 0V. Label the bounce region.

---

## SLIDE 30: Hardware Debouncing — RC Low-Pass Filter

**Content:**
- **Solution:** Add a resistor-capacitor (RC) low-pass filter to the switch output.
- **Circuit:** Pull-up resistor (e.g., 10kΩ) from Vcc to the input pin. Capacitor (e.g., 100nF) from the input pin to ground. Switch from the input pin to ground.
- **How it works:** When the switch bounces, the capacitor smooths out the rapid voltage changes. The RC time constant (τ = R × C) determines how fast the output responds. Choose τ > bounce duration.
- **Example:** R = 10kΩ, C = 100nF → τ = 1ms. This smooths bounces shorter than ~1ms.
- Alternative: software debouncing (ignore changes within a lockout period), but hardware is more reliable.

**Visual:** Schematic: Vcc → 10kΩ resistor → junction point → MCU input pin. From junction point: 100nF capacitor → GND. From junction point: switch → GND. Below: two voltage-time graphs side by side — left showing raw bounce, right showing the smoothed RC output.

---

## SLIDE 31: Digital Sensor Confidence — Majority Vote (Mode Average)

**Content:**
- **Purpose:** For binary (proximity) sensors, a single read may be a glitch. Read the sensor `n` times at the same position and take the **most common value** (the mode).
- **Algorithm:**
  1. Buffer `n` sensor readings (e.g., n = 7 or 8).
  2. Set `compare_value` = first reading in buffer. Set `counter` = 0.
  3. Iterate through buffer: for each reading that matches `compare_value`, increment `counter`.
  4. If `counter ≥ n/2`: **majority found** — accept `compare_value` as the true reading.
  5. If `counter < n/2`: set `compare_value` = next unmatched reading. Reset `counter` = 0. Repeat from step 3.
- **Key point:** This filters out occasional glitchy reads. If 5 out of 8 reads say "wall", then "wall" is the accepted answer.

**Visual:** Flowchart of the algorithm: Start → Load buffer → Set compare = buffer[0], count = 0 → Loop through buffer: if match, count++ → Decision: count ≥ n/2? → Y: Accept compare value → N: Set compare = next unmatched value, count = 0, repeat.

---

## SLIDE 32: Majority Vote — Worked Example

**Content:**
- **Buffer (8 readings):** `0001110, 0001100, 0001110, 0001100, 0001100, 0001100, 0101101, 0001110`
- **Round 1:**
  - Compare value = `0001110` (first reading)
  - Scan buffer: matches at positions 0, 2, 7 → counter = 3
  - Is 3 ≥ 8/2 = 4? **NO** — not a majority.
- **Round 2:**
  - Compare value = `0001100` (first unmatched reading, position 1)
  - Scan buffer: matches at positions 1, 3, 4, 5 → counter = 4
  - Is 4 ≥ 4? **YES** — majority found!
- **Result:** Accepted pattern = `0001100`
- The outlier `0101101` (position 6) is treated as noise and discarded.

**Visual:** Table showing the buffer as 8 rows. Round 1: rows matching `0001110` highlighted in blue (positions 0, 2, 7). Counter = 3. Red "X" — not majority. Round 2: rows matching `0001100` highlighted in green (positions 1, 3, 4, 5). Counter = 4. Green "✓" — majority found. Row 6 (`0101101`) crossed out as noise.

---

## SLIDE 33: Analogue Smoothing — Mean and Median

**Content:**
- For **distance sensors** (analogue output), use averaging to smooth noisy readings.
- **Mean Average:**
  - Sum all readings, divide by count.
  - Fast to compute. Affected by outliers (one extreme reading skews the average).
  ```c
  char mean(char buffer_size, int *buffer) {
      int accum = 0;
      for (char i = 0; i < buffer_size; i++) {
          accum += buffer[i];
      }
      return (char)(accum / buffer_size);
  }
  ```
- **Median Average:**
  - Sort all readings. Take the middle value.
  - Slower (requires sorting step). Robust to outliers — a single extreme value doesn't affect the median.
  ```c
  char median(int buffer_size, char *buffer) {
      sort(buffer_size, buffer);  // sort ascending
      return buffer[(buffer_size - 1) / 2];  // middle element
  }
  ```
- **Running average:** For slow-response sensors where taking `n` readings at one position isn't feasible. Maintain a rolling buffer and recalculate as new readings come in.
- **LSB truncation:** Before buffering, ignore the last few least-significant bits of the ADC value to filter high-frequency noise.

**Visual:** Block diagram: "Raw sensor data (noisy)" → splits into two paths → Path 1: "Mean function" (sum/divide) → "Smoothed output". Path 2: "Median function" (sort, pick middle) → "Smoothed output". Below, show a noisy signal graph and the smoothed result for each method, highlighting that mean is pulled by the outlier while median is not.

---

## SLIDE 34: Nyquist Sampling & The Oversampling Danger

**Content:**
- **Nyquist Theorem applied to spatial sampling:** To correctly detect a wall feature (e.g., the wall itself, or a gap), you must sample at least **twice per feature width**.
- **Oversampling danger:** If the spatial sampling rate is too high (reading sensors too frequently as the robot moves forward), a narrow crack in the wall will appear as a wide gap, potentially causing a false "turn" state.
- **Undersampling danger:** If the spatial sampling rate is too low, a narrow gap (actual turn opening) will be missed entirely.
- The correct sampling rate depends on:
  - `d_step` — linear distance per motor step
  - Wall width / gap width
  - Acceptable spatial resolution
- **Rule of thumb:** Sample at least twice per wall width (Nyquist), but not much more frequently.

**Visual:** Three diagrams of a robot moving past a wall with a small crack:
1. "Correct sampling" — 2 samples per wall width, crack is correctly identified as a crack (narrow).
2. "Oversampling" — 10 samples per wall width, crack produces multiple "no wall" readings and appears as a wide gap → false turn.
3. "Undersampling" — 0.5 samples per wall width, actual gap is missed entirely.
Mark sample points as dots along the robot's path.

---

# SECTION 9: TIMING MECHANISM — MOTOR STEPPING & SENSOR SAMPLING
*Justification: The master prompt has extensive content on tick counting, motor speed formulas, choosing 'm', and worked examples. This deserves a dedicated multi-slide section, not a single slide.*

---

## SLIDE 35: The Master Tick Concept

**Content:**
- Use a **Ticker object** to generate a master timing interrupt at a fixed period (the "tick").
- Everything is driven from this tick:
  - Motor stepping: toggle motor clock pin every `n` ticks.
  - Sensor reading: read sensors every `m × n × 2` ticks.
- **Critical constraint:** The execution time of the Ticker ISR handler must be **shorter** than the Ticker period. Otherwise, ticks will be missed.
- Using tick counting rather than changing the Ticker frequency allows speed changes without reconfiguring the timer hardware.

**Visual:** Horizontal timeline with evenly-spaced tick marks. Below: rows showing motor toggles (every n ticks) and sensor reads (every m×n×2 ticks). Annotate the Ticker period.

---

## SLIDE 36: Motor Speed Formula

**Content:**
- A stepper motor takes one step for each complete clock cycle (two toggles — HIGH then LOW).
- If the motor is toggled every `n` ticks, one full step takes `2 × n` ticks.
- **Linear speed formula:**

$$V = d_{step} \times \frac{f_{tick}}{2 \times n} \quad \text{(m/s)}$$

  - `d_step` = linear distance per motor step (m)
  - `f_tick` = Ticker frequency (Hz)
  - `n` = number of ticks per toggle
  - Factor of 2: two toggles per step

- **Calculating d_step:**

$$d_{step} = \frac{2 \pi r}{360} \times \theta_{step}$$

  - `r` = wheel radius
  - `θ_step` = step angle in degrees

**Visual:** Diagram showing the relationship: Ticker tick → count n ticks → toggle motor pin → count n more ticks → toggle again → 1 complete step = 2n ticks. Below, the formula with each variable labelled.

---

## SLIDE 37: Sensor Sampling Frequency Formula

**Content:**
- If sensors are read every `m` motor steps:
  - Each motor step = `2 × n` ticks
  - So sensors are read every `m × 2 × n` ticks
- **Sensor sampling frequency:**

$$f_{sample} = \frac{f_{tick}}{m \times 2 \times n} \quad \text{(Hz)}$$

- **Spatial resolution** of sensor reads:

$$\Delta x = m \times d_{step} \quad \text{(metres)}$$

  - This is the forward distance the robot travels between consecutive sensor reads.
  - Higher `m` = coarser spatial resolution but fewer CPU cycles spent on sensor processing.
  - Lower `m` = finer resolution but risk of oversampling.

**Visual:** Formula diagram with all variables labelled. Below, a number line showing the robot's position with markers every `m × d_step` indicating where sensor reads occur.

---

## SLIDE 38: Differential Steering via Tick Counting

**Content:**
- **Steering correction** is achieved by stepping one motor faster than the other:
  - **Steer Left:** Right motor steps more frequently than left motor (or left motor skips steps).
  - **Steer Right:** Left motor steps more frequently than right motor.
  - **Straight:** Both motors step at the same rate.
- Example from lectures: Left motor toggled every tick, right motor toggled every 2 ticks → left motor steps **2× faster** → robot curves to the right (steer-correct right).
- The tick counter for each motor can be independent, allowing fine-grained speed control.

**Visual:** Timing diagram with two rows:
- Row 1: "Left motor clock" — toggles every 1 tick (fast square wave).
- Row 2: "Right motor clock" — toggles every 2 ticks (slower square wave, half the frequency).
- Annotation: "Left motor 2× faster → robot steers right".
Below, a top-down view of the robot showing the curved path resulting from differential speed.

---

## SLIDE 39: Choosing 'm' — Front Sensor Sampling Rate (Worked Example)

**Content:**
- **Goal:** Choose `m` so the front sensor reads frequently enough to detect walls (Nyquist criterion).
- **Rule:** `m × d_step < ½ × wall_width`
- **Given values:**
  - Wheel diameter = 400/π mm → radius = 200/π mm
  - Wall width = 16 mm
  - Full stepping mode
  - Step angle = 0.9°
- **Calculate d_step:**
  - `d_step = (2 × π × 200/π) / 360 × 0.9 = (400/360) × 0.9 = 1 mm`
- **Calculate steps per wall width:**
  - `16 mm / 1 mm = 16 steps`
- **Apply Nyquist (minimum 2 samples per wall width):**
  - `m = 16 / 2 = 8 steps`
- **Result:** Read front sensor every **8 motor steps** = every `8 × 2 × n` ticks.

**Visual:** Diagram of the robot approaching a wall. The wall is 16mm wide. Along the robot's path, 16 step marks are shown (1mm apart). Every 8th step is circled and labelled "sensor read". Two sensor reads fall within the wall width. Annotation: "Nyquist satisfied: 2 samples per wall width."

---

## SLIDE 40: Choosing 'm' — Side Sensor Sampling Rate (Worked Example)

**Content:**
- **Goal:** Choose `m` for side sensors based on required spatial resolution and acceptable drift tolerance.
- **Given:**
  - Minimum spatial resolution needed = 6 mm (want to detect a 6mm change in side wall presence)
  - `d_step` = 1 mm
- **Calculation:**
  - `m = 6 mm / 1 mm = 6 steps`
- **Result:** Read side sensors every **6 motor steps** = every `6 × 2 × n = 12n` ticks.
- **Note:** Side sensors and front sensors can have different `m` values. Front sensor may need more frequent reads (smaller `m`) because walls ahead must be detected with enough stopping distance.
- **Oversampling warning:** If `m` is too small (e.g., m=1, reading every single step), cracks and imperfections in walls will trigger false state changes. The robot will appear "hesitant" — constantly correcting.

**Visual:** Side view of robot travelling along a wall. Sample points marked every 6mm along the path. Show a small crack in the wall — with m=6, the crack falls between two sample points and is correctly ignored. With m=1, multiple samples fall on the crack and flag it as "no wall".

---

# SECTION 10: STEPPER MOTOR HARDWARE
*Justification: Kept from pre-planned slides (L297, DS2003, motor types). This is hardware examinable content.*

---

## SLIDE 41: Unipolar vs Bipolar Stepper Motors

**Content:**
- **Unipolar motor (6-wire or 5-wire):**
  - Has a centre tap on each coil winding.
  - Only half the coil is energised at any time.
  - Simpler drive circuitry (no H-bridge needed).
  - Lower torque because only half the copper is active.
- **Bipolar motor (4-wire):**
  - No centre tap. The full coil is energised.
  - Requires H-bridge or full-bridge drive circuitry to reverse current direction.
  - Higher torque — all coil turns contribute to the magnetic field.
  - More complex driver but better performance per motor size.
- **For micro-mouse:** Bipolar motors are preferred for higher torque-to-weight ratio, but unipolar motors are simpler to drive.

**Visual:** Two side-by-side coil diagrams:
- Left: "Unipolar (6-wire)" — two coils, each with a centre tap connected to Vcc. Arrows showing current flowing through only half the coil at a time.
- Right: "Bipolar (4-wire)" — two coils, no centre tap. Arrows showing current flowing through the entire coil. Label: "Full coil energised = more torque".

---

## SLIDE 42: The L297 Stepper Motor Controller

**Content:**
- The L297 is a dedicated stepper motor controller IC that generates the correct coil energisation sequence.
- **Key control pins:**
  - **CW/CCW** — Direction control. 0V = clockwise, 5V = counter-clockwise (or vice versa depending on wiring).
  - **CLK** — Step clock input. Each rising (or falling) edge advances the motor by one step. The frequency of CLK determines the motor speed.
  - **H/F** — Half/Full step select. HIGH = half-stepping (finer resolution, smoother motion), LOW = full-stepping (faster, simpler).
  - **ENABLE** — Enables/disables the motor outputs.
  - **RESET** — Resets the internal state to a known starting position.
- The L297 outputs the correct 4-phase drive signals to the motor coils via a power driver (e.g., DS2003 or L298).
- The MCU only needs to provide CLK, CW/CCW, and H/F — the L297 handles the complex sequencing.

**Visual:** Pin diagram of the L297 IC. Highlight the MCU-connected pins (CLK, CW/CCW, H/F) on the left side. Show the motor coil drive outputs on the right side. Annotate each pin with its function.

---

## SLIDE 43: The DS2003 Darlington Driver

**Content:**
- The L297's outputs cannot directly drive motor coils — they don't provide enough current.
- The **DS2003** contains 7 Darlington transistor pairs that provide **current amplification** (current gain).
- Each Darlington pair takes a low-current logic signal from the L297 and drives the high-current motor coil.
- **Key feature:** Each channel includes an internal **flywheel (freewheeling) diode** for back-EMF protection (see next slide).
- The DS2003 can handle currents up to ~500mA per channel (check datasheet for exact ratings).

**Visual:** Internal schematic of one DS2003 channel: Input pin → base of first transistor → collector/emitter connected to base of second transistor → emitter to ground → collector to output pin (motor coil). Show the internal flywheel diode connected in parallel with the output (cathode to Vcc/COM, anode to output).

---

## SLIDE 44: Back-EMF Protection — Flywheel Diodes

**Content:**
- **Problem:** When current through a motor coil is suddenly switched off, the coil's inductance causes a large voltage spike (back-EMF). This spike can destroy the driver transistors.
- Formula: `V = -L × (di/dt)` — the faster the current change, the higher the spike voltage. Can reach hundreds of volts.
- **Solution:** Flywheel (freewheeling) diodes provide a safe path for the decaying coil current.
- The DS2003's **COM pin** is connected to the positive motor supply (e.g., 12V). The internal diodes connect from each output to COM, clamping the back-EMF spike to the supply voltage + one diode drop.
- **Critical wiring:** If the COM pin is left unconnected, the internal diodes have no reference and back-EMF protection fails — the DS2003 will be damaged.

**Visual:** Schematic showing: Motor coil connected between DS2003 output and 12V supply. When the transistor turns off, the coil current flows through the internal flywheel diode (shown in red) from the output pin to the COM pin (connected to 12V). The back-EMF spike is clamped. Show the voltage waveform: without diode (large spike) vs with diode (clamped).

---

# SECTION 11: KINEMATICS & TURNING
*Justification: Kept from pre-planned slides. Essential for understanding physical constraints.*

---

## SLIDE 45: Wheelbase and Differential Steering

**Content:**
- **Wheelbase (track width):** The distance between the left and right wheels, measured between their ground contact points.
- The wheelbase determines the **turning radius** of the robot.
- **Differential steering:** By running the left and right motors at different speeds:
  - Both same speed → straight line.
  - One faster → robot curves toward the slower wheel.
  - One stopped → robot pivots around the stopped wheel (minimum turning radius = wheelbase/2).
  - One forward, one reverse → robot spins on the spot (turning radius = 0, but swept path radius ≠ 0).
- The **axle midpoint** is the centre of rotation when spinning on the spot.

**Visual:** Top-down view of the robot chassis. Two wheels shown with the wheelbase dimension labelled. Three sub-diagrams showing: (1) Both wheels forward → straight arrow. (2) Left wheel faster → curve right. (3) Wheels opposite directions → spin on spot with rotation point at chassis centre.

---

## SLIDE 46: Motor Stepping Calculations (Worked Example)

**Content:**
- **Given:**
  - Linear speed desired: V = 3 cm/s
  - Step angle: θ = 1.8°
  - Wheel radius: r = 3/π cm
- **Calculations:**
  - Wheel circumference: `circ = 2 × π × r = 2 × π × 3/π = 6 cm`
  - Revolutions per second: `V / circ = 3 / 6 = 0.5 rev/s`
  - Degrees per second: `0.5 × 360 = 180 °/s`
  - Steps per second: `180 / 1.8 = 100 steps/s = 100 Hz`
  - Step period: `1 / 100 = 10 ms`
  - Distance per step: `d_step = circ × (θ / 360) = 6 × (1.8/360) = 0.03 cm = 0.3 mm`
- **Result:** To achieve 3 cm/s, the motor must be stepped at **100 Hz** (10ms period).

**Visual:** Formula chain shown step-by-step with each intermediate result. Diagram of a wheel showing: circumference arc, step angle, and the linear distance per step (arc length) at the wheel rim.

---

## SLIDE 47: U-Turn Corridor Constraints — Swept Path

**Content:**
- When the robot spins on the spot (both wheels turning in opposite directions), the **swept path** is the circle traced by the outermost point of the chassis.
- **Swept path radius `d`:** The distance from the centre of rotation (axle midpoint) to the farthest corner or protrusion of the robot.
- For the robot to perform a U-turn inside a maze corridor of width `W_c`:
  - **Constraint:** `W_c > 2 × d` (the corridor must be wider than the full diameter of the swept circle).
  - If `W_c ≤ 2 × d`, the robot will collide with the walls during the turn.
- **Solutions if the robot is too wide:**
  - Design a narrower robot (smaller wheelbase, compact body).
  - Perform a 3-point turn instead of a spin (drive forward, reverse at an angle, drive forward again).
  - Use the alternative approach: stop, reverse to the previous cell, then turn.

**Visual:** Top-down diagram showing the robot in a corridor. The robot is spinning on one wheel (or on its axle midpoint). A dashed circle shows the swept path radius `d`. The corridor width `W_c` is labelled. Two scenarios: (1) `W_c > 2d` — robot fits, green ✓. (2) `W_c ≤ 2d` — robot corner hits wall, red ✗.

---

# SECTION 12: POWER SUPPLY & BATTERIES
*Justification: Kept from pre-planned slides. Calculation-heavy examinable content.*

---

## SLIDE 48: Linear Voltage Regulators — LM7805

**Content:**
- The robot has two voltage domains:
  - **High voltage (12V):** Motor coils (via L297/DS2003).
  - **Low voltage (5V):** Microcontroller, sensors, logic ICs.
- The **LM7805** is a linear voltage regulator that converts 12V → 5V.
- **Operating principle:** The LM7805 dumps excess voltage as heat. It is essentially a controlled resistor.
- **Power wasted as heat:**

$$P_{waste} = (V_{in} - V_{out}) \times I_{total}$$

  - Example: `P = (12V - 5V) × 0.5A = 3.5 Watts`

- The LM7805 requires input/output decoupling capacitors (typically 100nF ceramic + 100μF electrolytic) for stability.

**Visual:** Block diagram: Battery (12V) → wire splits → Path 1: directly to motor driver (12V). Path 2: LM7805 → 5V rail → MCU, sensors. Show decoupling capacitors on input and output of the LM7805. Label the power dissipation arrow (heat).

---

## SLIDE 49: Heat Dissipation Calculation (Worked Example)

**Content:**
- **Given:**
  - V_in = 12V, V_out = 5V
  - Total 5V current draw: I = 300mA = 0.3A
  - LM7805 thermal resistance (no heatsink): R_θ = 65 °C/W
  - Ambient temperature: T_amb = 25°C
  - Maximum junction temperature: T_max = 125°C
- **Calculate power dissipated:**
  - `P = (12 - 5) × 0.3 = 2.1 W`
- **Calculate junction temperature:**
  - `T_junction = T_amb + (P × R_θ) = 25 + (2.1 × 65) = 25 + 136.5 = 161.5°C`
- **Result:** 161.5°C **exceeds** the 125°C maximum! The LM7805 will overheat and shut down (or be destroyed).
- **Solution:** Add a heatsink. Heatsink thermal resistance ≈ 5°C/W.
  - `T_junction = 25 + (2.1 × 5) = 25 + 10.5 = 35.5°C` — well within safe limits.
- **Alternative:** Use a switching regulator (more efficient, less heat).

**Visual:** Thermometer graphic showing temperature scale from 0°C to 150°C. Mark: T_amb = 25°C (green), T_junction without heatsink = 161.5°C (red, above the 125°C danger line), T_junction with heatsink = 35.5°C (green). Formula breakdown shown step by step.

---

## SLIDE 50: Battery Runtime Calculations

**Content:**
- Battery capacity is rated in mAh (milliamp-hours) at a specific discharge rate.
- **Runtime formula (linear approximation):**

$$Runtime = Rated\ Time \times \frac{Rated\ I}{Demand\ I}$$

- Example:
  - Battery rated: 2000mAh at 1A discharge (rated time = 2 hours at 1A)
  - Actual demand: 0.5A
  - Runtime = 2 × (1 / 0.5) = **4 hours**
- **Important:** This is an approximation. Real battery discharge is non-linear (Peukert's law). Higher discharge rates reduce effective capacity.
- Always include a safety margin — design for 80% of calculated runtime.

**Visual:** Graph showing battery voltage vs time for different discharge rates (0.5A, 1A, 2A). The 0.5A curve lasts longer but all curves show the characteristic discharge shape (flat plateau then rapid drop). Mark the "cutoff voltage" line.

---

## SLIDE 51: Battery Safety — Series vs Parallel

**Content:**
- **Series connection:** Batteries connected end-to-end. Voltages add, capacity stays the same.
  - Example: 3 × 4V cells in series = 12V, same mAh rating.
  - **Safe** as long as all cells are the same type and charge level.
- **Parallel connection:** Batteries connected side-by-side (positive-to-positive, negative-to-negative). Voltage stays the same, capacity adds.
  - Example: 3 × 4V 2000mAh cells in parallel = 4V, 6000mAh.
  - **DANGEROUS** — strongly discouraged for the micro-mouse project.
- **Why parallel is dangerous:**
  - If cells have slightly different voltages (even 0.1V difference), a large current flows between them trying to equalise — this can cause overheating, swelling, venting, or fire.
  - If one cell develops an internal short, the other cells dump their energy into it — fire/explosion risk.
  - Uneven aging causes progressive imbalance.
- **Rule:** Always use **series** connections. If you need more capacity, use a single larger cell.

**Visual:** Two diagrams:
- Left: "Series (SAFE ✓)" — Three battery icons stacked end-to-end with voltage labels adding up.
- Right: "Parallel (DANGEROUS ✗)" — Three battery icons side by side with a red warning symbol. Show a red arrow indicating "equalisation current" flowing between cells of different voltage. Include a fire/warning icon.

---

# SECTION 13: MBED C++ PROGRAMMING
*Justification: Merges pre-planned MBED slides with the master prompt's detailed code examples for Timer, Ticker, InterruptIn, and Ping/Echo.*

---

## SLIDE 52: Speed Signal Methods — Overview

**Content:**
- Four possible methods to generate the motor stepping clock signal:

| Method | Type | Pros | Cons |
|--------|------|------|------|
| **PwmOut** | Hardware PWM | Frees CPU entirely; very precise | Cannot count pulses sent to motor; need wheel encoder for distance |
| **Ticker** | Interrupt | Good precision; CPU free between ticks; can count ticks | Don't change Ticker period on the fly; use tick counting to vary speed |
| **Timer** | Polling | Simple to implement | May miss exact timing; main loop must keep checking |
| **Wait** | Blocking | Easiest to code | Worst timing; blocks CPU; affected by interrupts and program flow |

- **Recommended approach:** Use a **Ticker** with tick counting. The Ticker fires at a fixed rate, and the ISR counts ticks to decide when to toggle each motor. This gives precise timing + speed flexibility + distance tracking.

**Visual:** Four-quadrant comparison grid. Each quadrant shows the method name, a small timing diagram (showing precision/jitter), and a CPU utilisation bar (PwmOut = 0%, Ticker = low, Timer = medium, Wait = high).

---

## SLIDE 53: Timer Class — API and Code Example

**Content:**
- The **Timer** class provides a software stopwatch.
- **API:**

| Function | Description |
|----------|-------------|
| `Timer` | Create Timer object |
| `start()` | Start the timer |
| `stop()` | Stop the timer |
| `reset()` | Reset to zero |
| `read()` | Elapsed time in seconds (float) |
| `read_ms()` | Elapsed time in milliseconds (int) |
| `read_us()` | Elapsed time in microseconds (int) |

- Based on a 32-bit counter, 1μs resolution, ~30 minute maximum period.
- **Code example — 10ms square wave using Timer (polling):**

```c
#include "mbed.h"
Timer t1;
DigitalOut speed(p7);

void sig(void) {
    speed = !speed;  // toggle output pin
}

int main() {
    t1.start();
    while(1) {
        if (t1.read_ms() > 5) {  // 5ms per half-period = 10ms full period
            sig();
            t1.reset();
        }
    }
}
```

- **Limitation:** The `if` check runs in the main loop — if other code takes a long time, the toggle will be delayed (jitter).

**Visual:** The code above in a syntax-highlighted block. To the right, a timing diagram showing the resulting 10ms square wave on pin p7. Annotation: "Polling — timing depends on main loop speed".

---

## SLIDE 54: Ticker Class — API and Code Example

**Content:**
- The **Ticker** class generates periodic interrupts at a specified interval.
- **API:**

| Function | Description |
|----------|-------------|
| `Ticker` | Create Ticker object |
| `attach(&func, seconds)` | Attach function, call every N seconds |
| `attach_us(&func, microseconds)` | Attach function, call every N microseconds |
| `detach()` | Stop calling the attached function |

- **Code example — 10ms square wave using Ticker (interrupt-driven):**

```c
#include "mbed.h"

Ticker time_up;
DigitalOut speed(p7);

void sig(void) {
    speed = !speed;  // toggle output pin — called by interrupt
}

int main() {
    time_up.attach(&sig, 0.005);  // call sig() every 5ms → 10ms period
    while(1) {
        // main loop is FREE for other tasks (sensor reading, state logic, etc.)
    }
}
```

- **Advantage over Timer:** The main loop is completely free. The toggle happens precisely every 5ms regardless of what the main loop is doing.
- **Key rule:** Do NOT change the Ticker period on the fly to change speed. Instead, use **tick counting** within the ISR (count `n` ticks before toggling).

**Visual:** The code above in a syntax-highlighted block. To the right, a timing diagram showing the precise 10ms square wave. Annotation: "Interrupt-driven — precise timing, CPU free". Below, a comparison: Timer approach (jittery waveform) vs Ticker approach (clean waveform).

---

## SLIDE 55: InterruptIn Class — API and Pin Support

**Content:**
- The **InterruptIn** class creates an interrupt input pin that calls a function when a rising or falling edge is detected.
- **API:**

| Function | Description |
|----------|-------------|
| `InterruptIn(pin)` | Create interrupt input on specified pin |
| `rise(&func)` | Attach ISR to rising edge (0→1 transition) |
| `fall(&func)` | Attach ISR to falling edge (1→0 transition) |
| `mode(PullUp/PullDown)` | Set internal pull-up or pull-down resistor |

- **Supported pins:** P5 to P30, **except P19 and P20** (these are used for I2C and cannot be used as interrupt inputs).
- **Use case for micro-mouse:** Measuring echo pulse duration for acoustic distance sensors (see next slide).
- **Warning:** Electrical glitches on the input pin can trigger false interrupts. Use hardware debouncing (RC filter) or software debouncing (ignore interrupts within a lockout period).

**Visual:** MBED pin diagram with P5–P30 labelled. P19 and P20 crossed out in red. Highlight a few pins (e.g., P9) with annotation "InterruptIn capable".

---

## SLIDE 56: Acoustic Range Finder — Ping/Echo (Full Code)

**Content:**
- An acoustic range finder works by sending a short ultrasonic pulse (ping) and measuring the time until the echo returns.
- **Distance = f(travel_time)** — distance is proportional to the round-trip time of the sound pulse.
- **Implementation:** Use `DigitalOut` for the ping trigger, `InterruptIn` for the echo detection, and `Timer` to measure the time between them.
- **Pin connections:** Ping → p6 (DigitalOut), Echo → p9 (InterruptIn).

```c
#include "mbed.h"
#include "dist_sensor.h"  // assumed library with calc_distance()

InterruptIn Echo(p9);     // echo input — interrupt on rising edge
DigitalOut Ping(p6);      // ping output — trigger pulse
Timer T1;                 // measures echo travel time

void ISR() {
    T1.stop();            // stop timer when echo arrives
}

int main() {
    _disable_irq();       // disable interrupts during setup
    Echo.rise(&ISR);      // attach ISR to rising edge of echo
    Ping = 0;             // ensure ping is LOW
    T1.start();           // start timer
    
    while(1) {
        wait(5);                              // wait between measurements
        _enable_irq();                        // enable interrupts
        T1.reset();                           // reset timer to zero
        Ping = 1;                             // send ping pulse
        wait_ms(1);                           // 1ms pulse width
        Ping = 0;                             // end ping
        wait(2);                              // wait for echo to return
        distance = calc_distance(T1.read());  // calculate distance from time
    }
}
```

**Visual:** Three-part visual:
1. **Timing diagram:** Ping line shows a short HIGH pulse. Echo line shows a delayed HIGH pulse. The time between them is labelled "travel_time". Formula: `distance = speed_of_sound × travel_time / 2`.
2. **Pin connection diagram:** MBED chip outline with p6 → Ping sensor trigger, p9 → Echo sensor output.
3. **Code** in syntax-highlighted block with annotations.

---

## SLIDE 57: State Machines & Interrupt Priority in Code

**Content:**
- In the micro-mouse, multiple inputs may need to be checked, but some have **higher priority** than others.
- **Example priority order:**
  1. Front wall detection (highest — must stop immediately to avoid collision).
  2. Side wall changes (medium — determines turns).
  3. Steering correction (lowest — fine-tuning trajectory).
- **Implementation options:**
  - **Nested IF statements:** Check highest-priority conditions first. Lower-priority checks only run if higher-priority conditions are not triggered.
  - **Interrupt-based:** Critical inputs (e.g., front wall emergency stop) use hardware interrupts (InterruptIn), which override normal polling.
  - **Combination:** Use interrupts for safety-critical inputs, polling for everything else.
- The cascading `update_state()` function (Slide 15) inherently implements priority — the order of checks determines priority.

**Visual:** Flowchart showing nested IF structure: "Front wall?" → Y: STOP immediately (highest priority). N: → "Side wall gone?" → Y: ABOUT_TO_TURN. N: → "Drift detected?" → Y: STEER_CORRECT. N: → "Continue FORWARD". An interrupt lightning bolt icon breaking into the flow at the "Front wall" check, labelled "Hardware interrupt — cannot be missed".

---

# SECTION 14: SYSTEM OVERVIEW, TASK BREAKDOWN & EXAM PREPARATION
*Justification: Closing section from master prompt's recommendations. Ties everything together.*

---

## SLIDE 58: Complete System Block Diagram

**Content:**
- The full micro-mouse system in one diagram:
  - **Microcontroller (MBED LPC1768)** — central processing unit.
  - **Stepper Motor Driver Stack:** MCU → L297 (controller) → DS2003 (Darlington driver) → Stepper Motors (left & right).
  - **Sensors:** IR proximity sensors (IS471F-based) × 7 → MCU GPIO inputs. Optional ultrasonic range finder (Ping/Echo) → MCU.
  - **Power Supply:** Battery pack → LM7805 (5V for logic) + direct 12V for motors.
  - **User Interface:** Start button (with debounce circuit) → MCU. LEDs for status indication → MCU.

**Visual:** Full hardware block diagram. Central rectangle: "MCU (MBED)". To the left: sensor blocks with arrows into MCU. To the right: L297 → DS2003 → motor blocks with arrows from MCU. Top: power supply block with 12V and 5V rails shown. Bottom: button/LED block. All connections labelled with signal names (CLK, CW/CCW, sensor pins, etc.).

---

## SLIDE 59: Complete Task Breakdown

**Content:**
- All software tasks in the micro-mouse system, organised hierarchically:
  1. **Firmware Initialisation** — Configure MCU: timers, GPIO pins, interrupts. Initialise variables: state, position, orientation, maps.
  2. **Motor Control & Pulse** — Turning (left, right, U-turn). Steering correction. Stopping. Acceleration/deceleration.
  3. **Sensor Handling** — Raw sensor read. Confidence algorithm (majority vote for digital, mean/median for analogue). Noise reduction (LSB truncation).
  4. **Next State Encoder** — Pattern matching via update_state(). Bitwise AND pre-filtering.
  5. **Navigation** — Position update (at cell centre). Orientation update (after turn). Landmark calibration.
  6. **Mapping** — Update wall map per cell. Re-run Lee's algorithm when new walls found.
  7. **Speed Run** — Follow descending Lee's numbers from start to target.
  8. **Debug / User Interface** — System status messages (via serial/UART or LEDs).

**Visual:** Hierarchical tree diagram. Root: "Micro-Mouse Software". First-level children: each of the 8 tasks above. Second-level children: sub-tasks listed under each. Use colour coding: blue for real-time tasks (motor, sensor), green for logic tasks (state, navigation), orange for application tasks (mapping, speed run), grey for support tasks (init, debug).

---

## SLIDE 60: Left-Hand-on-Wall Algorithm — Rules and Limitations

**Content:**
- **Rules:** Always keep the left wall within reach. At every cell:
  - If you CAN turn left → turn left.
  - Else if you CAN go forward → go forward.
  - Else if you CAN turn right → turn right.
  - Else → U-turn (dead end).
- **When it works:** Simply connected mazes (all walls are connected to the outer boundary). The algorithm will eventually visit every reachable cell.
- **When it fails:** Mazes with "islands" (walls not connected to the boundary). The robot can circle an island forever without reaching the target.
- **Role in the project:** Stage 2 uses this algorithm for exploration without mapping. Stage 3 may use it as the exploration strategy during the mapping phase (combined with Lee's algorithm for pathfinding).

**Visual:** Two maze diagrams:
1. "Simply connected maze (works ✓)" — Robot traces the left wall, path visits all cells, reaches target.
2. "Maze with island (fails ✗)" — Robot loops around the central island indefinitely, never reaches target. The island is highlighted in red.

---

## SLIDE 61: Exam Tips and Common Mistakes

**Content:**
- **Calculation questions:** Practice these until they are automatic:
  - Motor stepping rate (steps/second) from desired speed, wheel radius, step angle.
  - Sampling rate 'm' from wall width and d_step (Nyquist).
  - LM7805 heat dissipation and heatsink requirement.
  - Battery runtime.
  - Swept path radius for U-turns.
- **Diagram questions:** Be able to draw from memory:
  - Top-level navigation state diagram (Slide 10).
  - Steering correction sub-state diagram (Slide 11).
  - Sensor bit layout (Slide 14).
  - L297/DS2003/motor connection block diagram.
- **Common mistakes:**
  - Confusing behavioural charts with flowcharts.
  - Forgetting the factor of 2 in motor speed calculations (two toggles per step).
  - Forgetting to update the adjacent cell's wall when recording a wall in the wall map.
  - Connecting batteries in parallel.
  - Leaving the DS2003 COM pin unconnected (no back-EMF protection).
  - Making the ISR longer than the tick period.

**Visual:** A "cheat sheet" style layout: key formulas in one column, key diagrams (miniature versions) in another, common mistakes with red warning icons in a third.

---

## SLIDE 62: Practice Questions

**Content:**
1. **State Diagram:** Draw the complete top-level navigation state diagram for the micro-mouse. Label all states and transitions.
2. **Timing Calculation:** A micro-mouse has wheels of radius 2/π cm and uses a step angle of 1.8°. Calculate the stepping rate needed for a speed of 5 cm/s. If a Ticker runs at 1000 Hz, how many ticks per toggle?
3. **Sampling Rate:** If d_step = 0.5mm and the maze walls are 12mm wide, calculate the maximum value of 'm' for the front sensor to satisfy Nyquist.
4. **Heatsink:** An LM7805 converts 9V to 5V with a load of 400mA. Ambient temperature is 30°C. Calculate the junction temperature without a heatsink (R_θ = 65°C/W). Is a heatsink needed?
5. **Bitwise Masking:** Given the sensor byte `0b01101001`, determine which mask(s) from {0x70, 0x07, 0x44} produce a non-zero result. What does each result indicate?
6. **Majority Vote:** Given the buffer `{0011, 0010, 0011, 0010, 0011, 0010, 0011}`, determine the majority vote result.

**Visual:** Questions laid out in numbered blocks, each with space for a worked answer below (or on a separate answer slide). Key formulas referenced for each question.

---

# END OF SLIDE PLAN
**Total: 62 slides across 14 sections.**
**Every slide includes: title, full content, and visual description.**
**Ready for direct slide deck construction.**
