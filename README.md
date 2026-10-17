Assignment 4: Facade ( Airport Operations )
Student: Alina Ibadulla | SE-2540 
Teacher: Software Design Patterns | Mektapbayeva Aruzhan 


This project implements the **Facade Design Pattern** over a three-stage transactional process for **Airport Operations**, adhering to all requirements of Assignment 4.


1. Overview & Architecture

The system orchestrates a 3-step boarding pipeline across three completely independent subsystems:
1. **Subsystem A (Intake / Check-In):** `CheckInValidationRules`, `PassengerRecord`, `CheckInRepository`
2. **Subsystem B (Processing / Security & Baggage):** `SecurityBaggageRules`, `BaggageRecord`, `BaggageRepository`
3. **Subsystem C (Finalization / Gate & Passport Control):** `PassportControlRules`, `GatePassRecord`, `GateRepository`

Facades
- **`PassengerBoardingFacade` (Client Facade):** Drives the primary $A \rightarrow B \rightarrow C$ workflow and handles two-depth automatic rollbacks when intermediate steps fail.
- **`AdminFacade` (Inspection Facade):** Allows system administrators to inspect the persisted state of any passenger across all subsystems and trigger manual compensations.

Durable State & Workflow Persistence
- Progress is persisted across application restarts using CSV storage (`data_subsystem_a.csv`, `data_subsystem_b.csv`, `data_subsystem_c.csv`, and `data_workflow.csv`).
- The entire chain's status is tracked via `BoardingWorkflow` (`WorkflowStatus`: `INITIATED`, `CHECKIN_COMPLETED`, `BAGGAGE_PROCESSING`, `BAGGAGE_COMPLETED`, `GATE_CLEARED`, `FAILED_AT_BAGGAGE`, `FAILED_AT_GATE`, `COMPENSATED`).


2. Principle of Least Knowledge (Law of Demeter)

Before (Without Facade - Direct Coupling & Client-Side Rollback)
Without a Facade, the client application must know about all three subsystems, manage individual repositories, and manually execute rollbacks when a step fails:

```java
// Client code directly coupled to 3 subsystems and manual rollback logic
CheckInRepository repoA = new CheckInRepository();
BaggageRepository repoB = new BaggageRepository();
SecurityBaggageRules rulesB = new SecurityBaggageRules();

PassengerRecord p = new PassengerRecord("P101", "KC123");
p.updateStatus(CheckInStatus.CHECKED_IN);
repoA.save(p);

if (!rulesB.inspectBaggage(40.0, false)) { // Baggage failed
    // Client must manually undo Subsystem A!
    p.updateStatus(CheckInStatus.CANCELLED);
    repoA.save(p);
}
