---
project_outline: "[[00-project/01-project-outline]]"
---

# Client-Side Prediction and Server Reconciliation in a Multiplayer Game

## 1. Actors and permissions
| Role | Needs | Responsibility / access |
| --- | --- | --- |
| Player | Responsive movement and consistent shared state | Connects a client and supplies movement input; cannot directly set authoritative position |
| Client | Display movement and synchronize with the server | Predicts local movement, buffers inputs, processes snapshots, and reconciles |
| Authoritative server | Maintain the official shared game state | Validates inputs, advances simulation, and broadcasts snapshots |
| Evaluator | Compare synchronization techniques fairly | Selects modes and network profiles, runs repeatable input traces, and exports measurements |

## 2. Use cases / user stories

### UC1: Start and join a session
- **Actor:** Player
- **Precondition:** The server is running and reachable.
- **Main flow:**
  1. The player starts a client and enters the server address.
  2. The server accepts the connection and assigns a player identifier.
  3. The client receives an initial authoritative snapshot.
  4. The client displays the environment and connected players.
- **Alternative / error flows:** Connection failure is displayed; input is disabled until initialization succeeds.
- **Postcondition:** The client has a valid starting state.

### UC2: Predict local movement
- **Actor:** Player, Client
- **Precondition:** The client is initialized and prediction is enabled.
- **Main flow:**
  1. The player supplies movement input.
  2. The client assigns a sequence number and buffers the input.
  3. The client applies the input using the shared movement model.
  4. The client sends the input to the server.
- **Alternative / error flows:** Invalid input is rejected; disconnected clients stop accepting session inputs.
- **Postcondition:** The client displays a predicted position without waiting for the server.

### UC3: Process authoritative movement
- **Actor:** Authoritative server
- **Precondition:** A connected client has supplied inputs.
- **Main flow:**
  1. The server checks player identity, sequence numbers, and permitted input values.
  2. It processes valid inputs according to the documented tick and delivery policy.
  3. It updates the authoritative state.
  4. It sends snapshots containing positions, server tick, and each player's last processed input sequence.
- **Alternative / error flows:** Duplicate or stale inputs are ignored; malformed messages are rejected.
- **Postcondition:** A new authoritative snapshot is available.

### UC4: Reconcile a predicted state
- **Actor:** Client
- **Precondition:** A newer snapshot has arrived and reconciliation is enabled.
- **Main flow:**
  1. The client records the difference between its state and the comparable server state.
  2. It removes buffered inputs acknowledged by the server.
  3. It restores the authoritative state for the local player.
  4. It replays remaining unacknowledged inputs in sequence.
  5. It applies the selected visual correction policy.
- **Alternative / error flows:** Older snapshots are ignored. Buffer overflow triggers an explicit resynchronization and is logged.
- **Postcondition:** Local simulation reflects the authoritative state plus pending inputs.

### UC5: Configure network conditions
- **Actor:** Evaluator
- **Precondition:** An experiment configuration is available.
- **Main flow:**
  1. The evaluator selects a delay, jitter, packet-loss rate, and random seed.
  2. The simulator applies the profile to both message directions.
  3. The system logs the configuration and actual delivery events.
- **Alternative / error flows:** Invalid profiles are rejected before the run starts.
- **Postcondition:** The run has a traceable network profile.

### UC6: Compare synchronization modes
- **Actor:** Evaluator
- **Precondition:** The baseline and experimental modes are implemented.
- **Main flow:**
  1. The evaluator runs a recorded input trace with server-confirmed movement.
  2. The same trace runs with prediction alone.
  3. The same trace runs with prediction and reconciliation.
  4. Each mode is tested under the same profiles and seed set.
  5. Results are exported for comparison.
- **Postcondition:** Measurements identify the benefits and costs of each technique.

### UC7: Recover after interruption
- **Actor:** Player, Client
- **Precondition:** A session experiences a prolonged interruption.
- **Main flow:**
  1. The client detects the documented timeout.
  2. It displays the disconnected state and clears stale session data.
  3. On reconnection, it obtains a fresh authoritative snapshot.
- **Postcondition:** The client resumes from a valid state rather than replaying inputs from the previous session.

## 3. Functional requirements
| Requirement | Acceptance criteria | Priority |
| --- | --- | --- |
| Support a shared session. | Two clients connect and receive both player states. | Must |
| Keep the server authoritative. | A client-provided position cannot overwrite the server's state. | Must |
| Use a fixed simulation timestep. | Movement advances using configured simulation ticks rather than render frame duration. | Must |
| Provide a server-confirmed baseline. | With prediction disabled, local movement is displayed after server confirmation. | Must |
| Predict local movement. | With prediction enabled, an accepted input updates local simulation on the next client simulation tick. | Must |
| Number and buffer inputs. | Each session's inputs have increasing sequence numbers and remain buffered until acknowledged or explicitly reset. | Must |
| Acknowledge processed inputs. | Snapshots include the last processed input sequence for the receiving player. | Must |
| Reconcile and replay inputs. | A newer snapshot restores authoritative state and replays only unacknowledged inputs. | Must |
| Handle stale and duplicate messages. | Older snapshots and duplicate processed inputs do not roll back or repeat movement. | Must |
| Simulate adverse networks. | Configurations alter delay, jitter, and loss in each direction and are recorded. | Must |
| Define delivery under packet loss. | Lost inputs are handled by a documented resend/redundancy and ordering policy; duplicated delivery never executes an input twice. | Must |
| Export experiment data. | Each run produces configuration, mode, seed, timestamp/tick, and metric records in CSV or JSON. | Must |
| Show diagnostic information. | The evaluator can inspect connection status, active mode, and current correction/error measurements. | Should |
| Compare visual correction policies. | Immediate correction and a configurable smoothing policy can run on identical traces. | Should |
| Display remote movement smoothly. | Optional interpolation uses received snapshots without affecting server authority. | Could |

Priority: **Must** = required for the final prototype; **Should** = important; **Could** = optional if time permits.

## 4. Business rules and constraints
| Rule or constraint | Rationale |
| --- | --- |
| The server alone decides authoritative state. | Prediction improves presentation but does not grant state authority. |
| Server and client use the same movement rules and timestep. | Reduces divergence caused by inconsistent simulation. |
| Input sequence numbers are scoped to a session. | Avoids confusing old messages with new-session inputs. |
| Replayed inputs produce no duplicated external side effects. | Reconciliation may execute the same movement calculation multiple times. |
| Visual smoothing does not modify authoritative simulation state. | Separates presentation from correctness. |
| Buffers and message sizes have defined limits. | Prevents unbounded growth during interruption or malformed traffic. |
| Comparisons use identical input traces and documented seed sets. | Keeps differences attributable to the synchronization mode. |
| Report configured one-way delay separately from measured round-trip time. | Avoids misleading latency comparisons. |

## 5. User interface and workflow
The prototype has a simple movement view, a connection panel, and an evaluator panel. The movement view displays the local player and remote players. An optional debug overlay shows predicted and authoritative positions. The evaluator panel selects synchronization mode, network profile, and correction policy, and starts or exports an experiment.

Typical workflow: start the server, connect two clients (UC1), move the local player (UC2–UC4), select network conditions (UC5), and run comparable recorded experiments (UC6).

## 6. Non-functional requirements
Numeric values below are proposed starting settings and will be reviewed after the first prototype.

| Requirement | How it will be verified |
| --- | --- |
| Local prediction avoids a network round-trip before movement is shown. | Measure input-to-visible-response time for each mode. |
| Reconciliation converges after input stops and delivery resumes. | Compare client/server positions at matching simulation ticks within a documented tolerance. |
| Two clients remain connected during a five-minute normal-network run. | Run a repeatable movement trace and check disconnect/error logs. |
| Experiments cover 0, 50, 100, and 200 ms configured one-way delay and 0%, 1%, and 5% loss. | Store and execute the defined profile matrix; add jitter profiles separately. |
| Results include at least five seeded runs per selected profile and mode. | Check run manifests and summarize median and high-percentile measurements. |
| Timing and position metrics compare equivalent events/states. | Use monotonic local timing and tick/input identifiers; avoid comparing unmatched current positions. |
| Experiments are reproducible. | Publish configuration, traces, seeds, code revision, runtime settings, and raw logs. |
| Components are maintainable. | Separate simulation, transport, network simulation, rendering, and measurement responsibilities. |

## 7. Open questions and risks
| Question / risk | Impact | Owner | Resolution / decision |
| --- | --- | --- | --- |
| Which language, framework, and transport should be used? | Affects development time and delivery behavior | Student | Confirm with the supervisor and build a small client-server spike before committing. |
| Client and server simulations may diverge. | Produces frequent corrections | Student | Share movement logic; verify timestep and floating-point tolerance. |
| Lost inputs or acknowledgements may leave pending inputs unresolved. | Causes divergence or buffer growth | Student | Specify sequencing, resend/redundancy, timeout, and resynchronization rules. |
| Smoothing can conceal error while adding visual lag. | Complicates interpretation | Student | Measure simulation error and displayed correction separately. |
| Latency metrics may compare different ticks or unsynchronized clocks. | Produces misleading results | Student | Match states using tick/input identifiers and use local monotonic timestamps. |
| Scope may grow into a full game. | Reduces time available for evaluation | Student | Keep basic movement and prioritize repeatable measurements. |
