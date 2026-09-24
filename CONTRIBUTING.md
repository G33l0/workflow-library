# Contributing

Thank you for contributing to the Senior Dev Workflow Library.

This repository is intended to contain reusable engineering workflows that help developers design reliable systems before implementation.

The objective is not to document only the successful path.

A useful workflow must explain what happens when the system is under stress, receives duplicate requests, encounters failures, or receives unexpected input.

---

# Contribution Principles

## 1. Document behavior before implementation

A workflow should describe what the system must do.

It should not depend on a specific:

- programming language
- framework
- database
- cloud provider
- payment provider
- frontend framework

Implementation-specific examples are acceptable when they clarify a principle, but the underlying workflow should remain reusable.

---

## 2. Do not document only the happy path

Every workflow should consider:

```text
Happy path
Duplicate request
Retry
Timeout
Network failure
Database failure
External provider failure
Concurrent request
Out-of-order request
Unauthorized request
Invalid state
Partial execution
Recovery
Reconciliation

⸻

Adding a Workflow

Create a Markdown file under the appropriate category.

Example:
workflows/04-payments/payment-reconciliation.md
Use lowercase kebab-case for filenames.

Good:
payment-reconciliation.md
wallet-withdrawal.md
subscription-renewal.md
admin-approval.md

Avoid:
PaymentReconciliationFinal.md
PaymentWorkflowV2.md
newWorkflow.md
workflow-final-final.md

⸻

Required Workflow Sections

A complete workflow should normally contain:
	1.	Purpose
	2.	Actors
	3.	Trigger
	4.	Inputs
	5.	Authoritative Data
	6.	Preconditions
	7.	States
	8.	State Transition Rules
	9.	Main Workflow
	10.	Idempotency
	11.	Concurrency
	12.	External Dependencies
	13.	Timeout Handling
	14.	Retry Handling
	15.	Duplicate Handling
	16.	Failure Handling
	17.	Reconciliation
	18.	Authorization
	19.	Security Considerations
	20.	Abuse Cases
	21.	Invariants
	22.	Audit Requirements
	23.	Observability
	24.	API Response Expectations
	25.	Test Cases
	26.	Recovery

A workflow may add additional sections when required.

⸻

The Five Failure Questions

Every significant workflow should answer these questions:

What happens when this happens twice?

Examples:
	•	payment request
	•	webhook
	•	refund
	•	reward claim
	•	order creation
	•	worker execution

The second execution must not create an unintended second business effect.

⸻

What happens when two requests happen simultaneously?

Examples:
Two withdrawals
Two purchases of the last item
Two coupon redemptions
Two reward claims
Two booking attempts
Two admin approvals
The workflow must define concurrency protection.

⸻

What happens when the network disappears?

Do not assume that:
No response = operation failed
The operation may have succeeded on the server.

⸻

What happens when an external provider succeeds but the application does not receive the response?

The workflow should normally enter:
PENDING
or:
UNKNOWN
until the operation can be reconciled.

⸻

What happens when the user is not authorized?

Authorization must be enforced on the server.

Do not rely on:
	•	hidden buttons
	•	disabled frontend controls
	•	hidden fields
	•	URL obscurity
	•	client-side role checks

⸻

Financial Workflow Requirements

Financial workflows require additional care.

At minimum, verify:
	•	amount is calculated server-side
	•	currency is authoritative
	•	customer identity is authoritative
	•	transaction ID is unique
	•	idempotency is implemented
	•	duplicate callbacks are handled
	•	ledger records are immutable where appropriate
	•	balance changes are atomic
	•	refunds are limited
	•	provider references are stored
	•	provider status is verified
	•	unknown states are reconciled
	•	concurrency is tested
	•	audit information is recorded

⸻

Security Requirements

Do not include secrets in workflow documentation.

Never document or commit:
API keys
Passwords
Private keys
Session tokens
Production credentials
Database passwords
Webhook signing secrets
Encryption keys
Use placeholders where examples require credentials.

Example:
PAYMENT_PROVIDER_SECRET
not:
sk_live_actual_secret_here

⸻

Testing Requirements

New workflows should include tests for:

Normal operation
Valid request
Expected state
Expected result

Duplicate operation
Same request twice

Retry
Request fails or times out
Client retries

Concurrency
Two or more requests simultaneously

Authorization
Wrong user
Wrong role
Wrong resource
Wrong state

External failure
Provider unavailable
Provider timeout
Provider returns failure
Provider returns unknown result

Database failure
Transaction rollback
Connection failure
Response lost after commit

Recovery
Interrupted operation
Reconciliation
Worker restart
Retry after failure

⸻

Review Checklist

Before submitting a workflow:
	•	Is the purpose clear?
	•	Are actors identified?
	•	Is authoritative state identified?
	•	Are states explicit?
	•	Are legal transitions explicit?
	•	Is authorization defined?
	•	Is idempotency defined?
	•	Is concurrency addressed?
	•	Are external dependencies addressed?
	•	Are timeout states defined?
	•	Is retry behavior defined?
	•	Is duplicate behavior defined?
	•	Is reconciliation defined?
	•	Are invariants defined?
	•	Are security considerations defined?
	•	Are abuse cases considered?
	•	Are audit requirements defined?
	•	Are observability requirements defined?
	•	Are failure tests defined?
	•	Is recovery defined?

⸻

Pull Requests

A pull request should clearly explain:
What workflow was added or changed?
Why is the workflow needed?
What failure cases are covered?
What security considerations were addressed?
What concurrency concerns were addressed?
What tests or validation were added?
Keep unrelated changes out of the pull request.

⸻

Quality Standard

A workflow should make another developer ask fewer questions, not more.

If the document leaves important behavior undefined, improve the workflow before considering it complete.