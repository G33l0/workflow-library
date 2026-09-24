---
# Architecture Principles

This document defines the architectural principles used throughout the Senior Dev Workflow Library.

The library is framework-agnostic.

The exact implementation may use different languages, frameworks, databases, queues, or cloud providers, but the underlying engineering principles should remain applicable.

---

# 1. Separate Responsibilities

A reliable application should separate major responsibilities.

A typical architecture may contain:

```text
Client
  |
  v
API / Application Layer
  |
  +------------------+
  |                  |
  v                  v
Domain Logic       Authorization
  |
  v
Persistence
  |
  +------------------+
  |                  |
  v                  v
Database           Cache
  |
  v
Events / Jobs
  |
  v
External Services

The exact architecture may vary.

The important requirement is that business-critical rules are not controlled exclusively by the client.

⸻

2. Client Layer

The client may:
	•	collect input
	•	display state
	•	request actions
	•	display validation errors
	•	optimistically update UI where appropriate
	•	retry requests

The client must not be the authoritative source for:
	•	balances
	•	permissions
	•	ownership
	•	prices
	•	payment status
	•	inventory
	•	approval status
	•	security decisions
	•	financial totals
	•	final workflow state

⸻

3. Application Layer

The application layer coordinates requests.

Typical responsibilities:
Authentication
Authorization
Validation
Business operation invocation
Transaction coordination
Response construction
The application layer should not blindly trust client-provided business state.

⸻

4. Domain Logic

Business rules should be explicit.

Examples:
An order may only be cancelled before shipment.
A withdrawal cannot exceed available balance.
A vendor cannot edit another vendor's product.
A refund cannot exceed the refundable amount.
A coupon can only be redeemed once per eligible customer.
A loan cannot be disbursed before approval.
These rules should exist independently of frontend behavior.

⸻

5. Persistence

The database normally acts as the authoritative source for application-owned state.

Examples:
User
Order
Payment
Wallet
Ledger
Inventory
Subscription
Booking
Loan
Installment
Vendor
Tenant

Database constraints should enforce important invariants where practical.

Examples:
UNIQUE payment reference
UNIQUE webhook event ID
UNIQUE idempotency key per operation scope
FOREIGN KEY ownership relationship
CHECK amount >= 0
Application-level validation alone is not sufficient for critical invariants when the database can enforce them.

⸻

6. External Systems

External systems may include:
	•	payment providers
	•	email providers
	•	SMS providers
	•	identity providers
	•	shipping providers
	•	storage services
	•	banking systems
	•	third-party APIs

Treat external systems as independent authorities with their own failure modes.

External systems can:
	•	timeout
	•	return errors
	•	return delayed responses
	•	return duplicate events
	•	return responses out of order
	•	succeed while the local request fails
	•	become temporarily unavailable

The architecture must account for these conditions.

⸻

7. External Side Effects

A major architectural rule:

Do not assume a database transaction automatically rolls back external side effects.

Example:
BEGIN TRANSACTION

Create payment record

Call external payment provider

Provider succeeds

Database transaction fails

ROLLBACK

The external provider may still consider the payment successful.

Therefore, external operations require:
	•	idempotency
	•	provider references
	•	durable operation records
	•	reconciliation
	•	retry strategy

⸻

8. Transaction Boundaries

Transactions should cover the local changes that must succeed or fail together.

Example:
BEGIN

Create ledger entry
Update wallet balance
Create financial event

COMMIT
The system should not intentionally create a state where:
ledger says +100
wallet says +0

unless the architecture explicitly models the difference.

See transactions.md.

⸻

9. Event-Driven Operations

Events can be useful for decoupling operations.

Example:
PaymentCompleted
       |
       +----> Update Order
       |
       +----> Grant Entitlement
       |
       +----> Send Notification
       |
       +----> Update Analytics
Events should be designed with:
	•	unique event IDs
	•	versioning where appropriate
	•	idempotent consumers
	•	retry handling
	•	dead-letter handling
	•	observability

⸻

10. Background Jobs

Background jobs are not guaranteed to execute exactly once.

A worker may:
	•	crash
	•	restart
	•	process the same job twice
	•	lose its network connection
	•	time out after the external operation succeeds

Therefore jobs should normally use:
job_id
idempotency
lease/lock
attempt count
retry policy
backoff
dead-letter/recovery path

⸻

11. Cache

Cache is generally not the authoritative source for critical business state unless the architecture explicitly makes it authoritative.

A cache may become:
stale
missing
incorrect
temporarily unavailable
Critical operations should use the authoritative source.

Example:
Cached balance

should not automatically determine whether a withdrawal is allowed.

⸻

12. Multi-Tenant Isolation

For multi-tenant applications, tenant identity should come from trusted server-side context.

Do not trust:
{
  "tenant_id": "customer-supplied-value"
}
without verifying that the authenticated actor belongs to that tenant and has access to the requested resource.

Every relevant database query should be scoped appropriately.

⸻

13. Resource Ownership

Resources should have explicit ownership or access relationships.

Example:
User
  |
  +---- owns ----> Order
  |
  +---- owns ----> Wallet
  |
  +---- belongs to ----> Organization
Authorization should verify the relationship before performing sensitive operations.

⸻

14. Observability

Important operations should be traceable.

Use appropriate:
request_id
correlation_id
operation_id
transaction_id
job_id
external_reference

These identifiers make it possible to follow a workflow across:
API
Database
Queue
Worker
External Provider
Webhook
Reconciliation

⸻

15. Architecture Decision Rule

When designing a workflow, ask:
These questions should influence the architecture before implementation begins.

