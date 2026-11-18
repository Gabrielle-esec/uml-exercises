# Hospital Management System — Design Decisions

This document explains the main UML modeling decisions
made for the Hospital Management System.

## 1. Inheritance

The system uses an abstract superclass called User.

Patient, Doctor, and Organizer inherit common properties:
- id
- name
- email
- passwordHash

They also inherit shared authentication behavior,
such as login() and logout().

This avoids code duplication and makes the system easier
to maintain.

## 2. Associations and Multiplicities

### Patient and HealthProblem

One patient can submit multiple health problems.
Each health problem belongs to exactly one patient.

Patient "1" -- "0..*" HealthProblem

### Doctor and Consultation

One doctor can conduct multiple consultations.
Each consultation involves exactly one doctor.

Doctor "1" -- "0..*" Consultation

### Patient and Consultation

One patient can attend multiple consultations.
Each consultation concerns exactly one patient.

Patient "1" -- "0..*" Consultation

### Order of Medical Consultations

A health problem can lead to consultations with
multiple doctors.

The Consultation class avoids duplicating patient
records and supports multiple medical opinions.

## 3. Health Problem Status

The system uses a ProblemStatus enumeration:

- PENDING
- IN_REVIEW
- RESOLVED

Each health problem records its creation and
last modification timestamps.

A separate history log would be required to
track every status change.

## 4. Prescription History

Each prescription is associated with a consultation.

A prescription may reference a previous prescription
to represent revisions.

Older prescriptions should be preserved rather
than overwritten.

## 5. Payment Strategy Pattern

Different payment methods require different
processing algorithms.

The Payment class delegates processing to the
PaymentStrategy interface.

Concrete implementations include:

- CashStrategy
- CheckStrategy
- CreditCardStrategy

This makes it possible to add new payment methods
without modifying the core Payment class.

## 6. Payment Data Integrity

To prevent incorrect or duplicate payments,
the system should implement:

- Unique transaction identifiers
- Idempotency keys
- Database transactions
- Payment status tracking
- Audit logs
- Reconciliation with external payment providers

A payment should not be transferred to a doctor
until the funds have been confirmed as received.

## 7. Design Assumptions

The current model makes several simplifying assumptions:

- Each consultation involves one patient and one doctor.
- A health problem can have multiple consultations.
- Each consultation produces at most one prescription.
- Each consultation has at most one associated payment.
- An organizer coordinates consultations and payments.
- A prescription can reference an earlier version.

These assumptions could be revised for a larger
real-world hospital information system.

## 8. UML Concepts Demonstrated

- Class diagrams
- Sequence diagrams
- Inheritance and generalization
- Associations and multiplicities
- Enumerations
- Interfaces and polymorphism
- Strategy design pattern
- Alternative and optional sequence flows
