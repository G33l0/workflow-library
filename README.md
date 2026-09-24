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

### 1. Server-authoritative state

The server determines the current state of a business resource.

The client may request a transition, but it cannot declare that the transition occurred.

Bad:

```text
POST /order/update

{
  "status": "paid"
}