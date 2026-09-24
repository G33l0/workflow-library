# Security Policy

## Purpose

The Senior Dev Workflow Library is intended to help developers design secure and reliable application workflows.

Security issues should be treated as engineering issues, not only as implementation details.

---

# Supported Versions

The latest version of the repository is the primary supported version.

Because this project is a documentation and workflow library, individual workflow documents may evolve independently.

---

# Reporting a Security Issue

If you discover a security issue in this repository, avoid publicly exposing sensitive information before maintainers have had an opportunity to review it.

Provide:

- affected file
- affected workflow
- description of the issue
- security impact
- reproduction steps where safe
- suggested mitigation where available

Do not include:

- real credentials
- private keys
- authentication tokens
- production secrets
- personal information
- private customer information

---

# Security Design Principles

## Server-side authorization

Authorization must be enforced by the server.

Client-side checks are useful for user experience but are not security boundaries.

---

## Least privilege

Users, services, administrators, and background workers should receive only the permissions required for their tasks.

---

## Explicit state transitions

Sensitive resources should not allow arbitrary client-selected states.

Example:

```text
PATCH /orders/123

{
  "status": "completed"
}
should not automatically be trusted.

Prefer explicit operations:
POST /orders/123/complete
with server-side validation of whether the transition is permitted.

⸻

Input validation

Treat external input as untrusted.

This includes:
	•	HTTP requests
	•	query parameters
	•	request bodies
	•	uploaded files
	•	webhook payloads
	•	third-party API responses
	•	imported data
	•	background job payloads
	•	message queue payloads

⸻

Authentication is not authorization

A valid authenticated user does not automatically have permission to access every resource.

Always consider:
Actor
Action
Target
Current State
Business Rules

⸻

Financial integrity

Financial operations require strong protection against:
	•	duplicate execution
	•	race conditions
	•	replay
	•	unauthorized access
	•	amount manipulation
	•	currency manipulation
	•	incorrect ownership
	•	duplicate webhooks
	•	provider mismatch
	•	partial updates
	•	incorrect refunds

⸻

Secrets

Never commit secrets to the repository.

Examples:
API keys
Passwords
Private keys
JWT signing secrets
Webhook secrets
Database credentials
Encryption keys
Cloud credentials
Use environment variables or a dedicated secrets-management system.

⸻

Logging

Logs should support investigation without exposing sensitive information.

Avoid logging:
	•	passwords
	•	authentication tokens
	•	full payment credentials
	•	private keys
	•	unnecessary personal data
	•	session cookies
	•	secret API credentials

⸻

Webhooks

Webhook consumers should normally verify:
	•	signature
	•	event authenticity
	•	event identifier
	•	expected provider
	•	expected resource
	•	expected amount where relevant
	•	expected currency where relevant
	•	current local state

Webhook processing should be idempotent.

⸻

File Uploads

File workflows should consider:
	•	file type validation
	•	size limits
	•	filename normalization
	•	storage isolation
	•	malware scanning where appropriate
	•	authorization
	•	access control
	•	path traversal
	•	content-type spoofing
	•	executable content
	•	download authorization

⸻

Rate Limiting

Sensitive operations should have appropriate limits.

Examples:
	•	login
	•	password reset
	•	OTP requests
	•	verification attempts
	•	withdrawals
	•	payment attempts
	•	coupon redemption
	•	messaging
	•	API requests

⸻

Auditability

Sensitive actions should be auditable.

Examples:
	•	administrator changes
	•	financial operations
	•	permission changes
	•	account recovery
	•	payout changes
	•	KYC decisions
	•	security setting changes

⸻

Vulnerability Classes to Consider

When reviewing a workflow, consider:
Broken authorization
Authentication bypass
Replay
Duplicate execution
Race conditions
Privilege escalation
Mass assignment
Input validation failures
Injection
Path traversal
Unsafe file handling
Information disclosure
Rate-limit bypass
Session abuse
Webhook spoofing
Insufficient auditability
Financial manipulation
State-transition bypass
Tenant isolation failure

⸻

Security Review Questions

Before production:
	•	Can an unauthorized user perform the operation?
	•	Can one user access another user’s resource?
	•	Can a user modify another tenant’s resource?
	•	Can the operation be replayed?
	•	Can it execute twice?
	•	Can two simultaneous requests bypass a limit?
	•	Can the client manipulate the amount?
	•	Can the client manipulate the state?
	•	Can a webhook be forged?
	•	Can an administrator exceed their permission?
	•	Can sensitive information appear in logs?
	•	Can rate limits be bypassed?
	•	Can an operation be recovered after partial failure?