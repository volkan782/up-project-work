---
name: volkan gulsoy
neptun: "c09332"
id: "2026-NP-01"
github: ""
---

# Client-Side Prediction and Server Reconciliation in a Multiplayer Game

Real-time multiplayer games must respond quickly to player input while keeping clients consistent with the server. Network delay and packet loss make this difficult: waiting for a server response can make movement feel slow, while immediately updating the client can produce differences between local and authoritative states. This project will build a small multiplayer movement prototype to implement and evaluate client-side prediction, server reconciliation, and state-error correction under simulated network conditions. The emphasis is on networking, synchronization, and measurable results rather than visual polish or complete game mechanics.

## Objectives
- **Primary objective:** Build an experimental client-server system and evaluate how prediction and reconciliation affect responsiveness and state consistency.
- **Target users / stakeholders:** Multiplayer game developers, researchers studying real-time synchronization, and players who expect responsive movement.
- **Measurable success criteria:**
  - One authoritative server supports at least two connected clients in a shared movement environment.
  - The local player moves immediately when prediction is enabled.
  - Clients reconcile server snapshots by restoring authoritative state and replaying unacknowledged inputs.
  - Experiments compare server-confirmed movement, prediction without reconciliation, and prediction with reconciliation.
  - Results report input-to-visible-response time, position error, correction magnitude, and network traffic under documented delay and loss settings.
- **Constraints:** A small desktop prototype; one shared movement model; controlled network simulation; reproducible experiments; technology choices confirmed during initial prototyping.

## Scope
### In scope
- A simple two-dimensional environment with basic movement and boundaries.
- An authoritative server with a fixed simulation timestep.
- Input sequence numbers, server acknowledgements, and state snapshots.
- Client-side prediction and replay of pending inputs.
- Server reconciliation and comparison of immediate versus smoothed visual correction.
- Configurable network delay, jitter, and packet loss.
- Experiment logs, analysis, and documentation.

### Out of scope
- A complete commercial game or polished artwork.
- Combat, matchmaking, accounts, and persistent worlds.
- Production-scale hosting and comprehensive anti-cheat protection.
- Advanced lag compensation for hit detection.

## Notes
The prototype is the main artefact, and controlled comparison is the main evaluation method. Network settings and numeric performance targets are initial experimental choices, not completed results. The first milestone is a reliable server-authoritative movement baseline; prediction and reconciliation will be added incrementally. The application topic identifier is 2026-NP-01. The GitHub field will be filled with the actual repository URL.
