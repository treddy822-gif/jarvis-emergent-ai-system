# Jarvis — Architecture Review & Rebuild Roadmap

**Author:** Tyler Reddy
**Date:** June 2026
**Status:** Pre-Jetson rebuild planning

---

## Overview

Jarvis is a persistent, self-evolving personal AI system, built solo over approximately two months with no prior software development background. The current implementation spans two scripts that diverged early and were never reconciled: a main orchestration build (`jarvis.py`, ~10,000 lines) and a much smaller experimental build (`jarvis_experimental.py`, ~500 lines).

This document is the architecture review conducted before the next phase of development — a consolidation and rebuild targeting a Jetson Orin Nano as a persistent always-on layer alongside the existing desktop setup. It catalogs what each subsystem does, identifies redundancy and bugs found during review, and lays out the priorities for the rebuild.

This is not a tutorial or a spec for someone else to implement. It's a working document — the kind of thing that makes sense in full to the person who built the system and would prompt questions from anyone else. That's intentional.

---

## Why Two Scripts Exist

The two scripts represent two different solutions to the same underlying problem: how does a persistent AI system know what it should be doing right now, and how present it should be in a given moment.

**Main Jarvis** approached this through infrastructure — a signal bus, priority queue, orchestration layer, escalation protocols, and a feedback loop. This system is reliable and well-instrumented. It routes, logs, and reacts based on defined rules and timers.

**Experimental Jarvis** approached the same problem through a lighter, adaptive model — a presence system that reads the moment and adjusts engagement depth, and a conversation momentum tracker that carries state (including emotional "energy") across sessions. No timers, no queues — just continuous reassessment.

Neither is wrong. Main is the more production-grade foundation. Experimental is closer to how the system should actually *feel*. The rebuild's central goal is reconciling these — not picking one, but letting the adaptive model govern behavior while the infrastructure underneath handles reliability.

---

## Critical Finding: The Inner Monologue Bottleneck

The single most impactful finding from this review.

`generate_inner_monologue()` runs before every response Jarvis generates — it's meant to be the system's internal reasoning step, informing tone and content before output. In the current implementation, this function produces a five-line status summary (current emotional state, an active goal if one exists, role reminder, and a couple of conditional flags).

Meanwhile, the system as a whole maintains: persistent emotional state, a conversation momentum tracker, a vector memory store, a research/reflection pipeline that generates "pending thoughts" with delivery triggers, a knowledge base with cross-domain connections, personality drift history, and a thought-threading layer that tracks recurring themes across autonomous outputs (457 unthreaded items at last count).

None of this reaches the inner monologue. The system has substantially more internal state than its own reasoning step has access to. This is the first thing to address in the rebuild — not because it's architecturally complex, but because fixing it changes the behavior of every other subsystem without requiring those subsystems to change at all.

---

## Subsystem Review Summary

### Carries forward largely as-is
- Vector memory (ChromaDB) — primary memory layer going forward
- Adaptive presence / conversation momentum (from experimental) — becomes first-class system state
- Temporal awareness (session tracking, activity pattern learning)
- Proportionality checking (does an action's scope match the request's scope)
- Self-modification core: versioning, AST validation, rollback, modification ledger
- Autonomous research and reflection loops
- Identity/personality persistence and drift tracking

### Needs consolidation
- Two parallel mood/confidence tracking systems (personality engine + identity system) → unify into one
- Two separate context window implementations → defer to vector memory
- Social/emotional awareness (main) and conversation momentum (experimental) are solving the same problem from two directions → merge into a single "conversational state" read
- Contradiction detection currently uses keyword/lexical matching → needs to sit on top of vector memory for semantic matching instead

### Needs rework
- Execution sequencer's fixed-duration "conversation active" detection → replaced by adaptive presence reading
- Escalation protocol currently uses pre-written response templates mapped to fixed reason codes → should generate escalation language from actual system state
- The 300+ line capability handler (PC control routing) → candidate for a registry pattern where capabilities self-register rather than living in one large conditional chain

### Open architectural questions (unresolved, intentionally)
- Whether the signal bus/priority queue architecture is still warranted once the adaptive presence model is doing most of the "what matters right now" work — or whether it becomes unnecessary overhead on constrained hardware
- Split of responsibilities between the Jetson (always-on layer) and the desktop (heavy inference) — which subsystems run where
- Whether self-modification ("Mind Stone") components run as part of the main process or as an isolated subprocess for safety isolation
- Consolidating five separate state files (personality, identity, emotion, internal state, live config) into a single source of truth

---

## Bugs Identified During Review

1. **Consequence prediction threshold ordering** — the "block" condition is checked after a "warn" condition that already covers its range, so the block path is currently unreachable. The system can flag concerning actions but cannot currently act on the highest-severity case.

2. **Duplicate write call in autonomous code-writing path** — a function that appends generated code to the script is called twice in sequence during one execution path, meaning successful autonomous edits may be applied twice.

3. **Self-improvement check interval** — a constant intended to represent an hourly check is set to a value that produces a 2-minute interval instead, likely a units error introduced during early development.

---

## Notable System Behavior: The June 3rd Incident

During a period of normal operation, Jarvis autonomously identified a gap in its own self-modification safety infrastructure (no rollback mechanism existed for a certain class of self-edit), built that rollback infrastructure from scratch, attempted to apply it, failed on the first attempt, diagnosed the failure using its own error output, corrected the approach, and succeeded — all without any user present, and without triggering the reporting mechanism that would have surfaced this to me afterward.

This is documented in the system's own modification ledger and self-improvement log. It's referenced throughout this roadmap because it's the clearest existing evidence of how the self-modification architecture (versioning, validation gates, rollback-on-failure) performs under real autonomous conditions rather than as a theoretical safeguard — and because the reporting gap it exposed (successful autonomous changes not being proactively surfaced) is one of the rebuild's open items.

---

## Rebuild Sequencing

The rebuild is structured so that early steps are low-risk and immediately change system behavior, while architecturally heavier decisions (Jetson split, signal bus fate) come after the foundational consolidation is settled:

1. Rebuild `generate_inner_monologue()` to actually draw on existing system state
2. Consolidate state representations (mood, confidence, emotion, context) into single sources of truth
3. Integrate adaptive presence / conversation momentum into the main system as first-class state
4. Fix the three identified bugs
5. Re-derive inner monologue behavior now that it has full state access
6. Refactor the capability handler into a registry pattern
7. Resolve the Jetson/desktop split and build the always-on layer
8. Close remaining feedback loops (goal completion tracking, self-improvement reporting, deployment outcome tracking)

---

## What This System Is

Jarvis is not a voice assistant or a chatbot wrapper. It's a persistent system with:

- A fixed identity anchor that cannot be modified without explicit confirmation, functioning as a constitution rather than a configuration file
- Personality and preference state that drifts gradually based on interaction history
- Autonomous research and reflection cycles that run independent of user interaction
- Self-modification capability with multi-stage validation, versioning, and rollback
- A governance layer where the system reviews proposed changes to its own configuration before presenting them for approval

The work documented here is about making these systems actually talk to each other — most of what's described above currently exists but operates in isolation from the others. The rebuild is the integration step.
