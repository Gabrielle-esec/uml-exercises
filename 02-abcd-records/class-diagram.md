# ABCD Records — Class Diagram

This UML class diagram models a mail-order system for CDs and tapes.

## Complete Class Diagram

```mermaid
classDiagram
direction TB

class Member {
    -int memberId
    -String name
    -String email
    -DateTime joinedAt
    +placeOrder() Order
    +viewOrders() List
}

class RoyalMember {
    -Decimal discountRate
    +applyDiscount() Decimal
    +orderUnavailableItem() void
}

class RegularMember {
    +placeOrder() Order
}

Member <|-- RoyalMember
Member <|-- RegularMember

class MembershipApplication {
    -int applicationId
    -String applicantName
    -String status
    +submit() void
    +approve() void
}

class Order {
    -int orderId
    -DateTime orderDate
    -Decimal totalAmount
    -OrderStatus status
    +calculateTotal() Decimal
    +confirmOrder() void
}

class OrderLine {
    -int quantity
    -Decimal unitPrice
    -Decimal discountApplied
    +calculateSubtotal() Decimal
}

class Item {
    <<abstract>>
    -int itemId
    -String title
    -Decimal price
    -int stockQuantity
    +checkAvailability() boolean
}

class CD {
    -String artist
    -String album
}

class Tape {
    -String artist
    -String album
}

Item <|-- CD
Item <|-- Tape

class Invoice {
    -int invoiceId
    -DateTime issuedAt
    -Decimal total
    +generate() void
}

class ShippingList {
    -int shippingListId
    -DateTime createdAt
    -String destination
    +generate() void
}

class Payment {
    <<abstract>>
    -int paymentId
    -Decimal amount
    -PaymentStatus status
    +processPayment() boolean
}

class CashPayment {
    +processPayment() boolean
}

class CheckPayment {
    -String checkNumber
    +processPayment() boolean
}

class BankDraftPayment {
    -String draftReference
    +processPayment() boolean
}

Payment <|-- CashPayment
Payment <|-- CheckPayment
Payment <|-- BankDraftPayment

class OrderProcessingClerk {
    -int clerkId
    -String name
    +verifyMembership() boolean
    +processOrder() void
}

class CollectionClerk {
    -int clerkId
    -String name
    +recordPayment() void
    +verifyPayment() boolean
}

class DailyRecord {
    -Date recordDate
    -int ordersProcessed
    -Decimal paymentsCollected
    +generateReport() void
}

class OrderStatus {
    <<enumeration>>
    PENDING
    PROCESSING
    BACKORDERED
    SHIPPED
    CANCELLED
}

class PaymentStatus {
    <<enumeration>>
    PENDING
    CLEARED
    FAILED
}

Member "1" --> "0..*" Order : places
MembershipApplication "0..1" --> "0..1" Member : may create

Order "1" *-- "1..*" OrderLine : contains
OrderLine "0..*" --> "1" Item : references

Order "1" *-- "0..1" Invoice : generates
Order "1" *-- "0..1" ShippingList : generates
Order "1" --> "0..*" Payment : paid by

OrderProcessingClerk "0..1" --> "0..*" Order : processes
CollectionClerk "0..1" --> "0..*" Payment : handles

DailyRecord "0..*" --> "0..*" Order : summarizes
DailyRecord "0..*" --> "0..*" Payment : summarizes

Order --> OrderStatus : uses
Payment --> PaymentStatus : uses
```

## Key UML Concepts

### Inheritance

RoyalMember and RegularMember inherit from Member.

CD and Tape inherit from the abstract Item class.

CashPayment, CheckPayment and BankDraftPayment
inherit from the abstract Payment class.

### Composition

An Order contains one or more OrderLines.

Each OrderLine belongs to exactly one Order.

An OrderLine references one Item and stores
the quantity and historical selling price.

### Associations and Multiplicities

- One member can place multiple orders.
- One order belongs to exactly one member.
- One order contains one or more order lines.
- One item can appear in multiple order lines.
- One order may generate an invoice.
- One order may generate a shipping list.
- One order can have multiple payments.
- One clerk can process multiple orders.

### Royal Membership

Royal members receive discounts and priority processing.

They can also order unavailable items.

Unavailable items can be marked as BACKORDERED
and fulfilled when stock becomes available.

### Payment Processing

Payments can be made using:
- Cash
- Check
- Bank draft

Payment status is tracked to avoid treating
an unconfirmed payment as completed.

## Modeling Assumptions

- Orders contain at least one item once confirmed.
- An order may have multiple payments.
- A pending order may not yet have an assigned clerk.
- A shipping list may be generated after payment confirmation.
- Daily records summarize orders and payments.
