---
layout: post
title: "Scaling with a Modular Monolith"
date: 2025-01-10 08:30:00 -0700
categories: [architecture, evolution]
tags: [monolith, modularity, scaling]
---

A modular monolith keeps deployment simple while still enforcing boundaries. It is often a better
first step than jumping directly to microservices because it avoids distributed systems overhead
while still organizing the system into clear, independent areas of responsibility.

## What makes it modular

- Clear module ownership and interfaces. Each module has a public surface area and hides its
  internals from other modules.
- Dependency rules enforced in code. The direction of dependencies is explicit and validated.
- Independent data access per module. Data is not shared casually; modules own their tables and
  expose data through their APIs.

## Why it scales before services do

Modularity lets teams move faster without paying the tax of network hops, distributed debugging,
and deployment coordination. You get:

- **Operational simplicity**: one deployable, one runbook, one set of dashboards.
- **Fast feedback**: local development and testing are easier and cheaper.
- **Clear seams**: later service extraction becomes a refactor, not a rewrite.

## Techniques to enforce boundaries

- Use internal packages or namespaces.
- Add lint rules or architectural tests to prevent cross-module imports.
- Treat database schemas as module-owned and expose data only via module APIs.
- Prefer explicit interfaces and DTOs over shared domain objects.
- Avoid shared "utility" modules that become dependency dumping grounds.

## Designing module APIs

Think of a module boundary like an internal service contract:

- Define a small set of entry points (commands, queries, events).
- Keep request/response models stable and version when necessary.
- Use domain events to notify other modules without creating tight coupling.

## Modular monolith sketch

<div class="mermaid">
flowchart LR
  subgraph Monolith[Modular Monolith]
    UI[Web UI]
    Order[Order Module]
    Catalog[Catalog Module]
    Billing[Billing Module]
  end

  UI --> Order
  UI --> Catalog
  Order -->|API| Billing
  Order -->|events| Catalog
  Order --> ODB[(Orders DB)]
  Catalog --> CDB[(Catalog DB)]
  Billing --> BDB[(Billing DB)]
</div>

## Signals to extract a service

- Independent scaling needs.
- Rapidly diverging release cadence.
- Operational constraints (latency, data locality).
- Compliance or isolation requirements (data residency, audit boundaries).
- Teams spending more time coordinating than delivering.

## Service extraction decision flow

<div class="mermaid">
flowchart TD
  Start[Module under sustained stress] --> Scale{Independent scaling needed?}
  Scale -- Yes --> Extract[Extract service]
  Scale -- No --> Cadence{Release cadence diverging?}
  Cadence -- Yes --> Extract
  Cadence -- No --> Ops{Operational constraints?}
  Ops -- Yes --> Extract
  Ops -- No --> Stay[Stay modular, reinforce boundaries]
</div>

## Maintenance areas that keep it healthy

Modular monoliths work well when you keep their boundaries and workflows clean. The following
areas are where entropy shows up first:

### Boundary enforcement

Maintain dependency rules in code and in CI. When modules can call each other freely, the system
quickly becomes a big ball of mud.

### Interface stability

Treat module APIs as contracts. Prefer additive changes, and deprecate carefully. If every change
breaks multiple modules, the boundary is too leaky.

### Data ownership discipline

Keep database ownership clear. No direct cross-module queries or shared tables without a formal
API. This prevents hidden coupling and unsafe migrations.

### Testing by module

Keep unit tests focused per module and add contract tests for module APIs. This keeps feedback fast
and makes refactors safer.

### Observability by module

Tag logs and metrics with module identifiers. This helps you isolate performance issues and spot
hotspots that indicate where to refactor or extract services.

### Dependency hygiene

Review shared libraries regularly. If a shared module grows without a clear purpose, it will pull
the system back into tight coupling.

## A steady path forward

Build the monolith with clean seams, measure where the pain is, and extract only when the cost of
coordination exceeds the cost of separation.
