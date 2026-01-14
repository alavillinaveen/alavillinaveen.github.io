---
layout: post
title: "Scaling with a Modular Monolith"
date: 2025-01-14 08:30:00 -0700
categories: [architecture, evolution]
tags: [monolith, modularity, scaling]
---

A modular monolith keeps deployment simple while still enforcing boundaries. It is often a better
first step than jumping directly to microservices.

## What makes it modular

- Clear module ownership and interfaces.
- Dependency rules enforced in code.
- Independent data access per module.

## Techniques to enforce boundaries

- Use internal packages or namespaces.
- Add lint rules to prevent cross-module imports.
- Treat database schemas as module-owned.

## Signals to extract a service

- Independent scaling needs.
- Rapidly diverging release cadence.
- Operational constraints (latency, data locality).

## A steady path forward

Build the monolith with clean seams, measure where the pain is, and extract only when the cost of
coordination exceeds the cost of separation.
