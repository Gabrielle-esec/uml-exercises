# ABCD Records — Sequence Diagrams

This document presents three UML sequence diagrams for
the ABCD Records mail-order management system.

## 1. Membership Verification and Order Placement

The system verifies membership before accepting an order.

```mermaid
sequenceDiagram
    autonumber
    actor M as Customer
    participant S as ABCD Records System
    actor C as Order Processing Clerk
    participant DB as Database

    M->>S: placeOrder(items)
    S->>C: notifyNewOrder()
    C->>S: verifyMembership(memberId)
    S->>DB: findMember(memberId)
    DB-->>S: membershipDetails

    alt Invalid or missing membership
        S-->>C: Membership invalid
        C-->>M: Send membership application
    else Valid Royal Member
        S-->>C: Royal membership confirmed
        C->>S: processOrder()
        S->>DB: saveOrder(PENDING)
        S-->>M: Order received
    else Valid Regular Member
        S-->>C: Regular membership confirmed
        C->>S: processOrder()
        S->>DB: saveOrder(PENDING)
        S-->>M: Order received
    end
```

## 2. Royal Member — Stock and Discount Processing

A Royal Member orders two different CDs.
One CD is available, while the other is out of stock.

Royal membership allows ordering unavailable items.

```mermaid
sequenceDiagram
    autonumber
    actor C as Order Processing Clerk
    participant S as ABCD Records System
    participant I as Inventory
    participant O as Order
    participant INV as Invoice
    actor M as Royal Member

    C->>S: processOrder(orderId)
    S->>O: getOrderLines()

    loop For each CD
        S->>I: checkAvailability(itemId)

        alt CD available
            I-->>S: In stock
            S->>I: reserveStock()
            S->>O: markLineReady()
        else CD unavailable
            I-->>S: Out of stock
            S->>O: markLineBackordered()
        end
    end

    S->>O: applyRoyalDiscount()
    S->>O: calculateTotal()
    O-->>S: discountedTotal

    S->>INV: generateInvoice()
    INV-->>S: invoice

    S-->>M: Send invoice
    S-->>M: Confirm available and backordered CDs
```

## 3. Check Payment and Shipping

The Royal Member pays by check.

The company waits for the check to clear before
shipping the available CD.

```mermaid
sequenceDiagram
    autonumber
    actor M as Royal Member
    actor CC as Collection Clerk
    participant S as ABCD Records System
    participant P as CheckPayment
    participant DB as Database
    actor OC as Order Processing Clerk

    M->>CC: Send check
    CC->>S: registerCheckPayment(orderId)
    S->>P: processPayment()
    P->>P: validateCheckDetails()
    P-->>S: Check recorded
    S->>DB: savePayment(PENDING)

    Note over S,DB: Waiting for bank clearance

    alt Check cleared
        CC->>S: confirmFundsReceived()
        S->>DB: updatePayment(CLEARED)
        S->>OC: notifyPaymentConfirmed()
        OC->>S: generateShippingList()
        S->>DB: saveShippingList()
        S-->>M: Confirm shipment of available CD
        Note over M,S: Second CD remains backordered
    else Check rejected
        CC->>S: reportPaymentFailure()
        S->>DB: updatePayment(FAILED)
        S-->>M: Request another payment
    end
```

## UML Concepts Demonstrated

- Actors and system participants
- Messages and return messages
- Alternative flows using alt and else
- Repeated actions using loop
- Conditional stock availability
- Royal Member discounts and backorders
- Payment status tracking

## Business Rules and Assumptions

1. Only valid members can place orders.
2. Royal Members receive discounts and priority.
3. Royal Members may order unavailable products.
4. Each CD is checked separately for availability.
5. Unavailable CDs are recorded as backordered.
6. Checks must clear before shipment.
7. Partial shipment is allowed in this model.
8. The remaining CD is shipped when available.

These diagrams are simplified educational models.
