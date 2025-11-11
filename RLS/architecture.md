# Rocket Launch System (RLS) - Architecture Document

## 1. Architecturally Important Aspects
The Rocket Launch System is a safety-critical system, so its most important design goals are:

- **Safety:** The system must never ignite accidentally. All failures must move the system to a safe state.
- **Reliability:** The communication link between the Control Unit (CU) and Launch Pad Unit (LPU) must handle delays or loss safely.
- **Correctness:** Commands must follow a strict sequence (Test → Enable → Ready → Launch).
- **Usability:** Lights and buttons must clearly show system status to avoid human mistakes.
- **Testability:** The system should allow simulation and diagnostics before real launches.

## 2. Proposed Solution
The system is divided into two subsystems:

- **Launch Pad Unit (LPU):** Controls the physical parts—battery, igniter, sensors, lights, and Test/Enable buttons.
- **Control Unit (CU):** Located at a safe distance. Sends commands (Ready, Launch) and shows status using LEDs.

**Design choices**
- Use a **Client–Server** structure: CU (client) sends commands; LPU (server) executes them.
- Each message requires an **acknowledgment**; if not received within a timeout, the CU retries or aborts.
- The LPU runs a **state machine** controlling safe transitions: Idle → Tested → Enabled → Armed → Ignited → Safe.
- Add **watchdog timers** so if communication fails, the LPU returns to Safe automatically.


## 3. Alternative Solutions
| Option | Why Not Chosen |
|--------|----------------|
| **Single monolithic system (no separation)** | Unsafe operator too close to the rocket. |
| **Peer-to-peer communication** | No clear authority; difficult to guarantee safety. |
| **Wireless-only without confirmation** | Could lose messages and trigger unsafe behavior. |

## 4. Architectural Patterns / Styles
- **Client–Server:** Separates user control from ignition hardware for safety.
- **Layered Architecture:** Hardware control, logic, and communication layers are independent.
- **State Machine:** Defines allowed transitions and ensures predictable, testable behavior.

## 5. Lightweight ADR – Communication Protocol
**ADR-001: Reliable Command Transmission**

- **Context:** Safe operation requires confirmed messages between CU and LPU.
- **Decision:** Use a message protocol with acknowledgments, retries, and timeouts.
- **Consequences:**
  - Increases safety and reliability.
  - Adds slight delay, acceptable for safe launches.
  - Enables recovery from communication loss.
