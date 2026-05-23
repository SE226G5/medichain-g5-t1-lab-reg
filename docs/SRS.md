# Software Requirements Specification (SRS)
## Module: LAB-REG — Specimen Registration & Barcoding
### Project: MediChain — نظام إدارة سلسلة المختبرات الطبية

---

## 1. Module Overview

This module handles specimen registration and barcoding within the MediChain system. It ensures every sample is uniquely identified, linked to the correct destination laboratory, and fully documented before moving to the next phase.

---

## 2. Team Members & Responsibilities

| Member Name | Role | Primary Responsibility |
|-------------|------|------------------------|
| حسام الحر | Student 1 (Leader) | Integration & Architecture, Component Diagrams, API Specs, Team Coordination |
| ناجي القاسم | Student 2 | Requirements & Analysis, Functional Requirements, Use Case Diagrams |
| يزن كحيلة | Student 3 | Process Modeling, Activity Diagrams, Business Rules Validation |
| يوسف بوظان | Student 4 | Data Design, ERD, Database Schema, Class Diagrams |

---

## 3. Functional Requirements

| ID | Requirement | Business Rule |
|----|-------------|---------------|
| FR-01 | System shall generate a composite specimen code (LabCode + TestType + Serial) | Code must be globally unique |
| FR-02 | System shall automatically assign specimen to destination lab based on test type | Assignment is automatic, no manual override |
| FR-03 | System shall block workflow if drawing timestamp is missing | Cannot proceed to next phase without timestamp |
| FR-04 | System shall validate fasting status for tests that require it | e.g., glucose test requires fasting = Yes |
| FR-05 | System shall print a barcode label after successful registration | Label includes composite code and patient info |
| FR-06 | System shall save registration to MediChain Core System | Data persisted after all validations pass |
| FR-07 | System shall log an error and block if patient status is invalid | e.g., patient not fasting for glucose test |

---

## 4. Non-Functional Requirements

| ID | Category | Requirement |
|----|----------|-------------|
| NFR-01 | Performance | Composite code generation in < 1 second |
| NFR-02 | Reliability | System available 99.9% of the time |
| NFR-03 | Security | All registration data encrypted at rest and in transit |
| NFR-04 | Auditability | All actions logged with user, timestamp, and action type |
| NFR-05 | Uniqueness | Composite code must be globally unique across all labs |

---

## 5. Use Cases

### Use Case Diagram
![Use Case Diagram](../images/use_case.png)
> Figure 1: MediChain LAB-REG Use Case Diagram

---

### UC-01: Register Specimen Data

| Field | Description |
|-------|-------------|
| **Actor** | Lab Receptionist / Technician |
| **Precondition** | Patient is registered in the system |
| **Main Flow** | 1. Enter patient info and test type → 2. Check fasting if required → 3. Generate composite code → 4. Assign destination lab → 5. Document drawing timestamp → 6. Print barcode → 7. Save registration |
| **Alt Flow** | If fasting not confirmed → system blocks and logs error |
| **Postcondition** | Specimen registered with unique code, status = Registered |

---

### UC-02: Validate Fasting Status

| Field | Description |
|-------|-------------|
| **Actor** | System (Automatic) |
| **Precondition** | Test type requires fasting (e.g., glucose) |
| **Main Flow** | 1. Check patient fasting flag → 2. If Yes → proceed → 3. If No → reject and log error |
| **Postcondition** | Registration blocked or allowed based on fasting status |

---

### UC-03: Generate Composite Code & Barcode

| Field | Description |
|-------|-------------|
| **Actor** | System (Automatic) |
| **Precondition** | Patient status validated successfully |
| **Main Flow** | 1. Combine LabCode + TestType + UniqueSerial → 2. Verify uniqueness → 3. Generate barcode → 4. Print label |
| **Postcondition** | Unique code assigned and barcode printed |

---

## 6. Process Modeling — Activity Diagram

![Activity Diagram](../images/activity.png)
> Figure 2: MediChain LAB-REG Activity Diagram

The activity diagram shows the complete workflow from receiving a patient test request to saving the registration. Key decision points include the fasting validation and drawing timestamp check.

---

## 7. Data Design

### 7.1 Entity-Relationship Diagram (ERD)

![ERD](../images/erd.png)
> Figure 3: MediChain LAB-REG ERD

### 7.2 Database Schema

| Table | Field | Type | Description |
|-------|-------|------|-------------|
| PATIENT | PatientID (PK) | INT | Unique patient identifier |
| PATIENT | Name | VARCHAR | Patient full name |
| PATIENT | FastingStatus | BOOLEAN | Yes/No — fasting confirmed |
| TEST_TYPE | TestTypeID (PK) | INT | Unique test identifier |
| TEST_TYPE | TestName | VARCHAR | Name of the lab test |
| TEST_TYPE | FastingRequired | BOOLEAN | Whether fasting is required |
| SPECIMEN_REGISTRATION | RegistrationID (PK) | INT | Unique registration ID |
| SPECIMEN_REGISTRATION | CompositeCode | VARCHAR | Unique: LabCode+Type+Serial |
| SPECIMEN_REGISTRATION | DrawingTimestamp | DATETIME | Exact time sample was drawn |
| SPECIMEN_REGISTRATION | Status | ENUM | Registered / Blocked / Cancelled |
| LABORATORY | LabID (PK) | INT | Unique lab identifier |
| LABORATORY | LabCode | VARCHAR | Lab short code |

---

## 8. Class Diagram

![Class Diagram](../images/class.png)
> Figure 4: MediChain LAB-REG UML Class Diagram

Main classes:
- **SpecimenRegistrationUI** — entry point for the technician
- **BarcodeService** — generates composite code and prints label
- **PatientStatusValidator** — validates fasting status
- **CoreSystemInterface** — communicates with MediChain Core
- **TestType, Laboratory, SpecimenRegistration** — data entities

---

## 9. Dynamic Modeling — State Machine Diagram

![State Machine](../images/state.png)
> Figure 5: MediChain LAB-REG Specimen Registration State Diagram

Specimen lifecycle:
`Created` → `DetailsEntered` → `StatusVerified` → `CodeGenerated` → `SampleDrawn` → `Registered ✓`

Edge cases:
- `ValidationBlocked` — patient not fasting
- `BlockedForNextPhase` — drawing timestamp missing
- `Cancelled` — error during save

---

## 10. Integration Points

| Direction | Module | Data Exchanged |
|-----------|--------|----------------|
| Inbound | Patient Registry | Patient info and fasting status |
| Outbound | Module 3 (INV-VAL) | Specimen ID + Test Type for inventory check |
| Outbound | Module 6 (LAB-TRK) | Specimen ID + Status for tracking |
| Outbound | MediChain Core | Full registration record saved |

---

## 11. Business Rules

| ID | Rule |
|----|------|
| BR-01 | Composite code must be unique: no two specimens share the same code |
| BR-02 | Drawing timestamp must be documented before proceeding to next phase |
| BR-03 | Tests requiring fasting must validate patient fasting status = Yes |
| BR-04 | Destination lab assignment is automatic based on test type |
| BR-05 | A blocked registration must be logged with reason and timestamp |
