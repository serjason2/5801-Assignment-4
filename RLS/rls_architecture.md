# Rocket Launch System (RLS) - Architecture Document

## 1. Architecturally Important Aspects
The Rocket Launch System is a safety-critical system, so its most important design goals are:

- **Safety:** The system must never ignite the rocket by accident. If something goes wrong, it should move to a safe state automatically.
- **Reliability:** Communication between the Control Unit (CU) and the Launch Pad Unit (LPU) must stay consistent even if there are small delays or dropped messages.
- **Correctness:** The system should only allow actions in the right order (Test → Enable → Ready → Launch).
- **Usability:** Lights and buttons must clearly show system status to avoid human mistakes.
- **Testability:** The system should allow simulation and diagnostics before real launches.

## 2. Proposed Solution
The system is divided into two subsystems:

- **Launch Pad Unit (LPU):** Manages the hardware like the battery, igniter, and indicator lights. It includes two buttons (Test and Enable).
- **Control Unit (CU):** Used by the operator from a safe distance. It has two buttons (Ready and Launch) and displays the status of the system.

**Design choices**
- Use a Client–Server structure: CU (client) sends commands; LPU (server) executes them.
- Each message requires an acknowledgment; if not received within a timeout, the CU retries or aborts.
- The LPU runs a state machine controlling safe transitions: Idle → Tested → Enabled → Armed → Ignited → Safe.
- Add watchdog timers so if communication fails, the LPU returns to Safe automatically.


## 3. Alternative Solutions
| Option | Why Not Chosen |
|--------|----------------|
| **Single monolithic system (no separation)** | Unsafe operator too close to the rocket. |
| **Peer-to-peer communication** | No clear authority; difficult to guarantee safety. |
| **Wireless-only without confirmation** | Could lose messages and trigger unsafe behavior. |

## 4. Architectural Patterns / Styles
- **Client–Server:** Separates the user’s control logic from the launch hardware, increasing safety.
- **Layered Architecture:** Divides the system into communication, logic, and hardware layers so that each can be developed and tested independently.
- **State Machine:** Makes the system’s behavior predictable and easier to verify through defined state transitions.

## 5. Lightweight ADR – Communication Protocol
**ADR-001: Reliable Command Transmission**

- **Context:** Safe operation requires confirmed messages between CU and LPU.
- **Decision:** Use a message protocol with acknowledgments, retries, and timeouts.
- **Consequences:**
  - Improves safety and reliability by preventing accidental commands.
  - Adds a small delay, which is acceptable for a safe launch sequence.
  - Allows the CU to recover or retry if messages are lost.
