

Because 2-phase commit solves atomicity by making the whole distributed system more tightly coupled and fragile.

The short version:

2PC gives you strong consistency, but you pay for it with latency, availability, complexity, and operational risk.

Let's build the intuition.

1. Start with one database

Suppose your Go service does:

BEGIN

    UPDATE accounts ...
    INSERT INTO transactions ...

COMMIT


Easy.

The database controls everything. If something goes wrong, it can roll back.

Go
 │
 ↓
PostgreSQL
 │
 ├── operation A
 ├── operation B
 └── COMMIT


Now imagine your operation involves two independent databases:

                  Go
               /     \
              ↓       ↓
        PostgreSQL    MySQL


You want:

PostgreSQL: COMMIT
MySQL:      COMMIT


or:

PostgreSQL: ROLLBACK
MySQL:      ROLLBACK


That's where 2PC comes in.

2. The problem with 2PC

The coordinator essentially does:

             Coordinator
                 │
          "Can you commit?"
             /       \
            ↓         ↓
           DB1       DB2

          "YES"     "YES"

             ↓
       "Okay, COMMIT"
             /       \
            ↓         ↓
          COMMIT    COMMIT


Sounds reasonable.

But consider this:

DB1: YES
DB2: YES

Coordinator: "COMMIT!"


Then the coordinator crashes.

             💥
        Coordinator
             X
            / \
           ↓   ↓
          DB1  DB2


The databases may have entered a prepared state but don't yet know whether they should commit.

They have to wait for the coordinator/recovery mechanism to tell them what happened.

That's one of the fundamental problems with 2PC.

3. It can hold resources for a long time

This is particularly important.

During 2PC, databases may have to keep locks/resources associated with the transaction while waiting for the final decision.

Imagine:

Transaction T1
    │
    ├── locks row A
    ├── locks row B
    │
    └── PREPARED
          ↓
       waiting...
          ↓
       waiting...
          ↓
       coordinator problem
          ↓
       waiting...


Meanwhile:

Transaction T2
       ↓
   wants row A
       ↓
     BLOCKED


A slow or unavailable participant can therefore cause other work to wait.

With a distributed system, "slow" is normal:

network latency
packet loss
database overload
machine restart
network partition
coordinator crash

So you're introducing distributed-system failure modes into your transaction mechanism.

4. Latency gets worse

A normal transaction might be roughly:

Go → DB
     ↓
   queries
     ↓
   COMMIT


With 2PC:

Go
 ↓
Coordinator
 ↓
DB1 ──┐
      ├── PREPARE
DB2 ──┘
 ↓
Coordinator
 ↓
DB1 ──┐
      ├── COMMIT
DB2 ──┘


You've added network round trips and coordination.

And your overall transaction latency is constrained by the slowest participant.

If:

DB1 = 10 ms
DB2 = 150 ms
DB3 = 500 ms


your distributed transaction has to deal with DB3.

5. Availability becomes harder

This is perhaps the biggest architectural concern.

Suppose:

Service
  │
  ├── DB A ✅
  │
  └── DB B ❌


With ordinary independent operations, perhaps you can continue operating against DB A.

With a distributed transaction:

DB A + DB B
     ↓
both required
     ↓
DB B unavailable
     ↓
transaction cannot complete


So you've coupled the availability of your systems.

This is especially undesirable in microservices.

6. Microservices make this even more painful

Imagine an e-commerce system:

Order Service
     │
     ├── Order DB
     │
     ├── Payment Service
     │       └── Payment DB
     │
     ├── Inventory Service
     │       └── Inventory DB
     │
     └── Shipping Service
             └── Shipping DB


You might initially think:

"Let's just put all of this inside one 2PC transaction."

So:

BEGIN
 ↓
create order
 ↓
charge payment
 ↓
reserve inventory
 ↓
create shipment
 ↓
COMMIT EVERYTHING


But now your order transaction depends on:

Order DB
Payment DB
Inventory DB
Shipping DB
Network
Payment service
Inventory service
Shipping service
Coordinator


That's a huge failure surface.

7. Instead, microservices often use a Saga

A common alternative is the Saga pattern.

Instead of one giant distributed transaction:

BEGIN
 ├── Order
 ├── Payment
 ├── Inventory
 └── Shipping
COMMIT


you have a sequence of local transactions:

Create Order
    ↓
Reserve Inventory
    ↓
Charge Payment
    ↓
Create Shipment


Each service commits its own database transaction.

If something fails, you perform a compensating action.

For example:

Create Order       ✅
Reserve Inventory  ✅
Charge Payment     ❌


Then:

Cancel Order
Release Inventory


Conceptually:

              Saga

Create Order ────────────────→ ✅

Reserve Inventory ───────────→ ✅

Charge Payment ──────────────→ ❌
                                  │
                                  ↓
                         Compensating actions
                                  │
                         Release Inventory
                                  │
                         Cancel Order


It's not the same as a database rollback.

That's important.

A rollback is:

"Pretend these database operations never happened."

A Saga is:

"The operations happened, so perform new operations that logically undo their effects."

8. The Outbox pattern is another important piece

There's another problem.

Suppose your Go service does:

BEGIN

INSERT order into DB

COMMIT

publish "OrderCreated" event


What if:

COMMIT ✅

then

Go crashes 💥

before publishing event


Now the order exists, but other services never hear about it.

The transactional outbox pattern addresses this.

Instead:

BEGIN

INSERT order

INSERT event into outbox table

COMMIT


Now both are in the same local transaction.

orders
----------------
id | customer
1  | Alice

outbox
-------------------------
id | event
1  | OrderCreated


Then a background worker publishes the outbox event:

Outbox
   ↓
Message broker
   ↓
Inventory Service
   ↓
Payment Service


This is extremely useful to understand if you're going into Go backend development.

9. So when is 2PC appropriate?

2PC isn't "bad."

It's useful when you genuinely need distributed atomicity and your infrastructure supports it reliably.

For example, some enterprise systems deliberately use distributed transactions because their requirements demand very strong transactional guarantees.

It's also worth learning because understanding 2PC teaches you a lot about:

distributed systems
failure modes
consistency
locking
coordination
network partitions
transaction protocols

But for many modern microservice architectures, you'd rather design the system so that you don't need a distributed transaction in the first place.

10. The architectural progression I'd recommend for you

Since you're currently learning Go database handling, I'd go through these concepts in this order:

                    Go + database
                         │
                         ↓
                database/sql
                         │
                         ↓
                  Transactions
                         │
                         ↓
             Commit / Rollback
                         │
                         ↓
             Isolation levels
                         │
                         ↓
                  Row locking
                         │
                         ↓
             Concurrent transactions
                         │
                         ↓
              Distributed systems
                         │
               ┌─────────┴─────────┐
               ↓                   ↓
              2PC                Saga
               │                   │
               │                   ↓
               │               Outbox
               │                   │
               └─────────┬─────────┘
                         ↓
                 Event-driven systems


And there's a really important Go exercise that ties all of this together:

Build a small bank transfer API with PostgreSQL:

POST /transfer

Alice ──₹1000──→ Bob


First implement it incorrectly without a transaction, then with a transaction, then introduce two concurrent transfers and see why isolation/row locking matters.

That progression will make transactions, SELECT ... FOR UPDATE, isolation levels, and eventually distributed transactions much easier to understand.