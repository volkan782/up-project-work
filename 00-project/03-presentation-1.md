---
project_outline: "[[00-project/01-project-outline]]"
---

<!-- 5xl -->
# Client-Side Prediction and Server Reconciliation

An experimental multiplayer game prototype under simulated network conditions

Author: volkan gulsoy  
Neptun: c09332  
Supervisor: Péter Novák
Project ID: 2026-NP-01

---

## Problem

- Multiplayer games exchange inputs and state over a network.
- Delay means server confirmation arrives after the player acts.
- Waiting for confirmation makes movement feel slow.
- Predicting immediately can produce differences from server state.
- Packet loss and jitter make synchronization harder.

---

## Proposed idea

- Build a small two-dimensional multiplayer movement prototype.
- Use one authoritative server and at least two clients.
- Predict the local player's movement immediately.
- Reconcile against server snapshots and replay pending inputs.
- Evaluate responsiveness, error, and correction under controlled conditions.

---

## Main goal: compare three modes

| Mode | What it answers |
| --- | --- |
| Server-confirmed movement | How responsive is movement when the client waits? |
| Prediction without reconciliation | How much does prediction help, and how does state diverge? |
| Prediction with reconciliation | Can responsiveness improve while correcting divergence? |

Same movement model, input traces, and network profiles.

---

## Target users

| Role | Needs | Interaction |
| --- | --- | --- |
| Player | Responsive movement | Connects and moves a character |
| Game developer | Understand synchronization tradeoffs | Inspects implementation and results |
| Evaluator | Reproducible comparisons | Configures profiles and exports measurements |

---

## Planned scope

**In scope**

- Basic movement and boundaries
- Fixed-timestep authoritative simulation
- Input sequencing and acknowledgements
- Prediction, reconciliation, and correction
- Simulated delay, jitter, and loss
- Experiment logs and analysis

**Out of scope**

- Full game mechanics and polished graphics
- Matchmaking and persistent worlds
- Production-scale hosting
- Hit-detection lag compensation

---

## User journey 1: joining a session

1. Start the authoritative server.
2. Connect two clients.
3. Receive player identifiers and initial snapshots.
4. Display both players in the shared environment.

**Result:** a working baseline for synchronization experiments.

---

## User journey 2: immediate movement

1. The player presses a movement key.
2. The client numbers and buffers the input.
3. The client predicts movement locally.
4. The input travels to the server.
5. The server processes it and acknowledges it in a snapshot.

**Result:** local movement begins without waiting for network confirmation.

---

## User journey 3: reconciliation

1. A newer authoritative snapshot arrives.
2. The client removes acknowledged inputs.
3. It restores the server's local-player state.
4. It replays the remaining pending inputs.
5. The view applies immediate or smoothed correction.

**Result:** the client corrects divergence while retaining pending movement.

---

## User journey 4: a controlled experiment

1. Select a synchronization mode and network profile.
2. Run a recorded movement trace with a stored seed.
3. Export timing, position, correction, and traffic measurements.
4. Repeat for the other modes and seeds.
5. Compare the results under equivalent conditions.

---

## Evaluation plan

- Starting delay settings: **0, 50, 100, 200 ms one way**
- Starting packet-loss settings: **0%, 1%, 5%**
- Additional profiles for jitter
- At least five seeded runs per selected profile and mode
- Metrics: input-to-visible-response time, matched-state position error, correction magnitude, and network traffic

These are planned experiments; no results are claimed yet.

---

## Next steps

- Confirm the implementation stack and scope with the supervisor.
- Review literature on prediction and reconciliation.
- Build the authoritative movement baseline.
- Add input sequencing, prediction, and replay.
- Validate packet-loss handling and measurement methods.
- Run experiments and discuss benefits and limitations.
