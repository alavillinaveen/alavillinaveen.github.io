---
layout: post
title: "Architecture Decision Records: Lightweight, Lasting Decisions"
date: 2025-01-12 09:00:00 -0700
categories: [architecture, process]
tags: [adr, decisions, documentation]
---

Architecture Decision Records (ADRs) are short, durable notes that explain why a system is the way
it is. They reduce churn in discussions, provide context for future changes, and keep teams aligned.

## Why ADRs work

- They capture **intent**, not just implementation.
- They are small enough to keep current.
- They make trade-offs explicit.

## A simple structure

Most ADRs can fit in five sections:

1. **Title**: A clear decision statement.
2. **Status**: Proposed, accepted, superseded, or deprecated.
3. **Context**: The forces, constraints, and facts.
4. **Decision**: What we chose and why.
5. **Consequences**: What gets easier or harder now.

## Maintenance tips

- Keep the record close to code.
- Link to diagrams or RFCs sparingly.
- Supersede, do not rewrite, when the decision changes.

### Sample snippet

```markdown
# ADR 012 - Prefer async messaging for order events

## Status
Accepted

## Context
Order creation must not block on downstream services.

## Decision
Publish order-created events to a durable broker.

## Consequences
Consumers must handle retries and idempotency.
```
