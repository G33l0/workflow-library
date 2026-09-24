# Senior Dev Workflow Library

A reusable engineering workflow library for building reliable, secure, observable, and failure-resistant applications.

This repository documents how important application workflows should behave before implementation begins.

It is designed to help developers think through business logic, state transitions, failure modes, security boundaries, concurrency, external dependencies, recovery, and testing before writing production code.

---

## Purpose

Many application failures are not caused by syntax errors or missing features.

They happen because the workflow was never fully defined.

Examples:

- A payment succeeds but the application does not receive the response.
- A webhook is delivered twice.
- Two requests withdraw the same balance simultaneously.
- A user retries an order after the first request already succeeded.
- A database commits successfully but the HTTP response is lost.
- An external provider returns an unknown status.
- A background worker processes the same job twice.
- A user attempts to modify a resource they no longer own.
- A frontend reports a state that does not match the server.
- An administrator performs an action that should require another administrator's approval.

This library exists to define those situations before they become production incidents.

---

## Core Principle

The frontend is not the source of truth.

The server owns authoritative business state.

This normally includes:

- identity
- authentication state
- authorization
- ownership
- prices
- balances
- inventory
- workflow state
- transaction state
- order state
- subscription state
- approval state
- entitlements
- limits
- security decisions
- financial records

Client input is treated as input, not truth.

---

## Universal Workflow Rule

Every important workflow should answer:

1. What starts the workflow?
2. Who is allowed to start it?
3. What resources are affected?
4. What data is authoritative?
5. What states can exist?
6. What transitions are legal?
7. What conditions are required for each transition?
8. What happens if the request is duplicated?
9. What happens if two requests happen simultaneously?
10. What happens if the network fails?
11. What happens if the database fails?
12. What happens if an external provider times out?
13. What happens if the external provider succeeds but the application does not receive the response?
14. How is an unknown operation reconciled?
15. What invariant must never be violated?
16. What must be recorded?
17. What should the user receive as the response?
18. How can the operation be retried safely?
19. How can the operation be recovered?
20. How is the workflow tested?

If these questions cannot be answered, the workflow is not fully designed.

---

## Engineering Principles

### 1. Server authoritative state

The server determines the current state of a business resource.

The client may request a transition, but it cannot declare that the transition occurred.

Bad:

```text
POST /order/update

{
  "status": "paid"
}

Better:
POST /orders/{id}/pay
The server verifies payment and determines whether the order may transition to PAID.

⸻

2. Explicit state machines

Important workflows should have explicit states and legal transitions.

Example:
PENDING_PAYMENT
      |
      v
PAID
      |
      v
PROCESSING
      |
      v
SHIPPED
      |
      v
DELIVERED
Invalid transitions must be rejected.

For example:
PENDING_PAYMENT -> DELIVERED
should not be possible unless the workflow explicitly defines such a transition.

⸻

3. Idempotency

Repeating the same operation must not create an additional business effect.

Examples:
	•	duplicate payment request
	•	duplicate webhook
	•	duplicate refund request
	•	duplicate withdrawal request
	•	duplicate reward claim
	•	duplicate background job
	•	duplicate order submission

The system should recognize the existing operation and return the existing result where appropriate.

⸻

4. Atomicity

Operations that must remain consistent should be committed atomically.

Example:
Create ledger entry
+
Update balance
must not leave the system in a state where one succeeds and the other does not.

⸻

5. Concurrency protection

Assume multiple requests can happen at the same time.

Never rely on:
if balance >= amount:
    subtract amount
without concurrency protection.

Use appropriate database transactions, locks, conditional updates, constraints, or other concurrency mechanisms.

⸻

6. Unknown is not failure

This distinction is critical.
FAILED
means the system has evidence that the operation failed.
PENDING
means the operation is still unresolved.
UNKNOWN
means the system cannot yet determine the final result.
SUCCESS
means the operation has been verified as successful.

A timeout does not automatically mean failure.

⸻

7. Reconciliation

When internal and external systems disagree, the application must have a reconciliation path.

Example:
Payment provider = SUCCESS
Application       = PENDING
The correct response is not automatically to create another payment.

The system should query the authoritative source, verify the operation, and apply the missing state transition exactly once.

⸻

8. Auditability

Important business operations should leave an audit trail.

An audit record may contain:
	•	actor
	•	action
	•	resource
	•	previous state
	•	new state
	•	timestamp
	•	request ID
	•	correlation ID
	•	result
	•	relevant external reference
	•	reason where applicable

Audit data should not contain secrets unnecessarily.

⸻

9. Least privilege

Every operation should require only the permissions necessary to perform that operation.

Authentication answers:
Who are you?
Authorization answers:
Can you perform this exact action
on this exact resource
in its current state?

10. Recovery must be designed

Failure handling is part of the workflow, not an afterthought.

A workflow should explain how the system recovers from:
	•	client retries
	•	network failures
	•	database failures
	•	worker crashes
	•	provider timeouts
	•	duplicate callbacks
	•	delayed webhooks
	•	partial execution
	•	stale state
	•	service outages

⸻

Repository Structure
senior-dev-workflow-library/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md
│
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── new-workflow.md
│   │   └── workflow-improvement.md
│   │
│   └── workflows/
│       └── markdown-check.yml
│
├── docs/
│   ├── architecture.md
│   ├── workflow-standard.md
│   ├── state-machines.md
│   ├── idempotency.md
│   ├── concurrency.md
│   ├── transactions.md
│   ├── reconciliation.md
│   ├── authorization.md
│   ├── audit-logging.md
│   ├── error-handling.md
│   ├── retry-strategy.md
│   ├── observability.md
│   └── testing-strategy.md
│
├── workflows/
│   ├── 01-authentication/
│   ├── 02-authorization/
│   ├── 03-account-management/
│   ├── 04-payments/
│   ├── 05-wallets/
│   ├── 06-installments/
│   ├── 07-loans/
│   ├── 08-banking/
│   ├── 09-subscriptions/
│   ├── 10-ecommerce/
│   ├── 11-marketplace/
│   ├── 12-orders/
│   ├── 13-inventory/
│   ├── 14-shipping/
│   ├── 15-refunds/
│   ├── 16-disputes/
│   ├── 17-bookings/
│   ├── 18-dating/
│   ├── 19-social/
│   ├── 20-messaging/
│   ├── 21-notifications/
│   ├── 22-saas/
│   ├── 23-multi-tenant/
│   ├── 24-admin/
│   ├── 25-kyc/
│   ├── 26-rewards/
│   ├── 27-referrals/
│   ├── 28-coupons/
│   ├── 29-content/
│   ├── 30-files/
│   ├── 31-search/
│   ├── 32-location/
│   ├── 33-fraud-risk/
│   ├── 34-background-jobs/
│   ├── 35-webhooks/
│   ├── 36-apis/
│   ├── 37-data/
│   ├── 38-support/
│   ├── 39-logistics/
│   ├── 40-ai-apps/
│   └── 41-iot/
│
├── templates/
│   ├── workflow-template.md
│   ├── state-machine-template.md
│   ├── api-workflow-template.md
│   ├── financial-workflow-template.md
│   ├── webhook-template.md
│   ├── background-job-template.md
│   └── security-workflow-template.md
│
└── checklists/
    ├── pre-development.md
    ├── pre-merge.md
    ├── pre-production.md
    ├── financial-feature.md
    ├── marketplace-feature.md
    ├── authentication-feature.md
    ├── admin-feature.md
    └── incident-review.md


⸻

Workflow Categories

Identity

Authentication, registration, sessions, account recovery, email verification, phone verification, password changes, and account security.

Authorization

Roles, permissions, ownership, resource-level access, approvals, and contextual authorization.

Finance

Payments, balances, transfers, refunds, withdrawals, installments, loans, payouts, and reconciliation.

Commerce

Cart, checkout, orders, inventory, shipping, returns, exchanges, discounts, and fulfillment.

Marketplace

Vendors, commissions, escrow, seller orders, vendor balances, and payouts.

Social

Dating, matching, blocking, messaging, notifications, moderation, and safety workflows.

SaaS

Subscriptions, organizations, tenants, plans, usage, quotas, billing, and entitlements.

Security

Rate limiting, risk signals, verification, abuse prevention, audit logging, and sensitive operations.

Infrastructure

APIs, webhooks, queues, background jobs, caching, external services, retries, and reconciliation.

⸻

Financial State Model

Financial workflows should distinguish at least:
CREATED
PENDING
PROCESSING
SUCCESS
FAILED
UNKNOWN
CANCELLED
REFUNDED
The exact states depend on the business operation.

Never assume:
HTTP timeout = payment failed
Instead:
HTTP timeout
     |
     v
UNKNOWN/PENDING
     |
     v
RECONCILIATION
     |
     +------ SUCCESS
     |
     +------ FAILED

⸻

Universal Workflow

A general workflow can be modeled as:
DEFINE
  |
  +-- authoritative state
  +-- actor
  +-- resource
  +-- legal states
  +-- legal transitions
  +-- invariants
  |
  v
REQUEST
  |
  +-- authenticate
  +-- authorize
  +-- validate
  +-- normalize
  +-- generate/use idempotency key
  |
  v
CHECK
  |
  +-- current state
  +-- ownership
  +-- permissions
  +-- limits
  +-- existing operation
  +-- concurrency conditions
  |
  v
EXECUTE
  |
  +-- transaction
  +-- conditional update/lock
  +-- external operation if required
  |
  v
RECORD
  |
  +-- state change
  +-- business event
  +-- audit record
  +-- external reference
  |
  v
VERIFY
  |
  +-- resulting state
  +-- invariant
  +-- external confirmation
  |
  v
RESPOND
Failure path:
FAILURE
   |
   +-- Is failure confirmed?
   |       |
   |       +-- YES -> FAILED
   |       |
   |       +-- NO -> UNKNOWN/PENDING
   |
   +-- Can operation be retried safely?
   |
   +-- Is reconciliation required?
   |
   +-- Can operation be recovered?

⸻

Definition of Done

A workflow should not be considered production-ready until it defines:
	•	Purpose
	•	Actors
	•	Inputs
	•	Authoritative data
	•	States
	•	Valid transitions
	•	Preconditions
	•	Authorization
	•	Idempotency
	•	Atomicity
	•	Concurrency behavior
	•	External dependencies
	•	Timeout behavior
	•	Retry behavior
	•	Duplicate handling
	•	Failure handling
	•	Reconciliation
	•	Invariants
	•	Audit requirements
	•	Observability
	•	Security considerations
	•	Abuse cases
	•	Test cases
	•	Recovery procedure

⸻

Documentation

Core engineering concepts:
	•	Architecture
	•	Workflow Standard
	•	State Machines
	•	Idempotency
	•	Concurrency
	•	Transactions
	•	Reconciliation
	•	Authorization
	•	Audit Logging
	•	Error Handling
	•	Retry Strategy
	•	Observability
	•	Testing Strategy

⸻

Contributing

See CONTRIBUTING.md.

The goal is not to collect simple coding recipes.

The goal is to document reliable business workflows that remain correct under failure, duplication, concurrency, malicious input, and partial system outages.

⸻

License

This project is licensed under the MIT License.

See LICENSE.
---
