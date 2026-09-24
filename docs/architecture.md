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