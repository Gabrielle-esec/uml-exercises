# Hospital Management System — Class Diagram

This UML class diagram models an online hospital management platform.

It covers:
- User roles and inheritance
- Patient health problems and consultations
- Medical prescriptions and their history
- Payment processing using the Strategy design pattern

## UML Class Diagram

```mermaid
classDiagram
direction TB

class User {
    <<abstract>>
    -int id
    -String name
    -String email
    -String passwordHash
    +login() boolean
    +logout() void
}

class Patient {
    +submitHealthProblem() HealthProblem
    +viewPrescriptions() List
}

class Doctor {
    -String specialty
    +reviewProblem() void
    +createPrescription() Prescription
}

class Organizer {
    +assignDoctor() void
    +sendPrescription() void
    +forwardPayment() void
}

User <|-- Patient
User <|-- Doctor
User <|-- Organizer

class HealthProblem {
    -int problemId
    -String description
    -ProblemStatus status
    -DateTime createdAt
    -DateTime updatedAt
    +updateStatus() void
}

class ProblemStatus {
    <<enumeration>>
    PENDING
    IN_REVIEW
    RESOLVED
}

class Consultation {
    -int consultationId
    -DateTime consultationDate
    -String status
    +schedule() void
    +complete() void
}

class Prescription {
    -int prescriptionId
    -String content
    -DateTime issuedAt
    +createRevision() Prescription
}

Patient "1" -- "0..*" HealthProblem : submits
Patient "1" -- "0..*" Consultation : attends
Doctor "1" -- "0..*" Consultation : conducts
HealthProblem "1" -- "0..*" Consultation : concerns
Organizer "0..1" -- "0..*" Consultation : coordinates

Consultation "1" -- "0..1" Prescription : produces
Doctor "1" -- "0..*" Prescription : writes
Prescription "0..*" --> "0..1" Prescription : revises
HealthProblem --> ProblemStatus : uses

class Payment {
    -int paymentId
    -Decimal amount
    -String status
    -DateTime paymentDate
    +processPayment() boolean
}

class PaymentStrategy {
    <<interface>>
    +process(amount: Decimal) boolean
}

class CashStrategy {
    +process(amount: Decimal) boolean
}

class CheckStrategy {
    +process(amount: Decimal) boolean
}

class CreditCardStrategy {
    +process(amount: Decimal) boolean
}

PaymentStrategy <|.. CashStrategy
PaymentStrategy <|.. CheckStrategy
PaymentStrategy <|.. CreditCardStrategy

Payment "0..*" --> "1" PaymentStrategy : uses
Patient "1" -- "0..*" Payment : makes
Consultation "1" -- "0..1" Payment : billed by
Organizer "0..1" -- "0..*" Payment : forwards
Payment "0..*" --> "1" Doctor : payable to
```

## Design Decisions

### Inheritance
Patient, Doctor and Organizer inherit common attributes and
authentication methods from the abstract User class.

### Consultations
The Consultation class allows a patient to consult multiple
doctors for the same health problem.

### Prescriptions
Each consultation can produce a prescription. Prescriptions
may reference earlier prescriptions to preserve revision history.

### Payment Strategy
The Strategy design pattern separates payment processing
from the Payment class.

Cash, check and credit card payments use different strategies.

### Multiplicities
- A patient can submit multiple health problems.
- A patient can attend multiple consultations.
- A doctor can conduct multiple consultations.
- A health problem can lead to multiple consultations.
- Each payment is associated with one consultation.
