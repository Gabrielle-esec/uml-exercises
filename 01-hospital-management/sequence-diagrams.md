# Hospital Management System — Sequence Diagrams

This document presents three UML sequence diagrams:
1. Patient registration
2. Medical consultation
3. Payment processing

## 1. Patient Registration

The organizer registers a new patient and creates their account.

```mermaid
sequenceDiagram
    autonumber
    actor P as Patient
    actor O as Organizer
    participant S as Hospital System
    participant DB as Database

    P->>O: Request registration
    O->>S: registerPatient(details)
    S->>S: validateDetails()
    S->>DB: checkExistingPatient()

    alt Valid new patient
        S->>S: generatePatientId()
        S->>DB: savePatient()
        DB-->>S: Registration confirmed
        S-->>O: Account created
        O-->>P: Send activation instructions
    else Invalid or duplicate patient
        S-->>O: Registration rejected
        O-->>P: Request corrected information
    end
```

## 2. Medical Consultation

A patient submits a health problem. The organizer coordinates
the consultation and sends the doctor's prescription.

```mermaid
sequenceDiagram
    autonumber
    actor P as Patient
    participant S as Hospital System
    actor O as Organizer
    actor D as Doctor
    participant DB as Database

    P->>S: submitHealthProblem(description)
    S->>DB: createHealthProblem(PENDING)
    DB-->>S: problemId
    S-->>P: Submission confirmed

    S->>O: notifyNewProblem()
    O->>S: createConsultation()
    S->>DB: updateProblem(IN_REVIEW)

    O->>D: requestConsultation()
    D->>S: reviewHealthProblem()
    S-->>D: Problem details

    D->>S: createPrescription()
    S->>DB: savePrescription()
    DB-->>S: Prescription saved

    S-->>O: Prescription ready
    O->>S: sendPrescription()
    S-->>P: Prescription available
    S->>DB: updateProblem(RESOLVED)
```

## 3. Payment Processing

The system delegates payment processing to a PaymentStrategy.

```mermaid
sequenceDiagram
    autonumber
    actor P as Patient
    participant S as Hospital System
    participant PS as PaymentStrategy
    participant DB as Database
    actor O as Organizer
    actor D as Doctor

    P->>S: selectPaymentMethod()
    P->>S: pay(consultationId, amount)
    S->>DB: createPayment(PENDING)

    S->>PS: process(amount)

    alt Cash
        PS->>PS: recordCashReceipt()
    else Check
        PS->>PS: registerCheck()
    else Credit Card
        PS->>PS: authorizeCardPayment()
    end

    PS-->>S: processingResult

    alt Payment accepted
        S->>DB: recordProcessingResult()
        S-->>P: Payment recorded

        opt Funds confirmed as received
            S->>DB: markPayment(CLEARED)
            S->>O: notifyFundsAvailable()
            O->>S: requestDoctorTransfer()
            S->>DB: recordTransfer()
            S-->>D: Transfer notification
        end
    else Payment rejected
        S->>DB: markPayment(FAILED)
        S-->>P: Payment failed
    end
```

## UML Concepts Used

- **Actors:** external users interacting with the system.
- **Lifelines:** participants in each interaction.
- **Messages:** requests and responses between participants.
- **alt:** alternative flows based on conditions.
- **opt:** optional behavior.
- **Strategy Pattern:** interchangeable payment algorithms.

## Assumptions

- The organizer is responsible for patient registration.
- A problem can involve multiple medical consultations.
- A prescription is made available after a consultation.
- Payment acceptance does not always mean funds have cleared.
- Funds must be confirmed before transferring money to a doctor.
