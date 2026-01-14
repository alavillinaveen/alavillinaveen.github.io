---
layout: post
title: "Event-Driven Architecture Without the Fairy Dust"
date: 2025-01-13 10:15:00 -0700
categories: [architecture, integration]
tags: [events, messaging, patterns]
---

Event-driven architecture (EDA) helps decouple services, but it introduces new failure modes. The
goal is not to add events everywhere, but to use them where they reduce coupling and improve flow.

## When EDA is a good fit

- Multiple consumers need the same facts.
- Work can happen asynchronously.
- You want to add new capabilities without changing a core service.

## Core building blocks

- **Event schema**: A contract with explicit versioning.
- **Broker**: Durable queue or log that supports replay.
- **Idempotent consumers**: Safe to run more than once.

## Common pitfalls

- Treating events as commands with hidden coupling.
- Ignoring replay and backfill.
- Forgetting that "eventual" includes monitoring and alerting.

## Practical heuristics

- Start with a single event stream and grow deliberately.
- Prefer facts over instructions in event payloads.
- Document consumer expectations near the publisher.
