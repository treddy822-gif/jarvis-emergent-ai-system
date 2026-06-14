Jarvis — A Persistent, Self-Modifying AI System

Overview

Jarvis began as a personal project inspired by the idea of a real-time assistant — something like the fictional JARVIS. What started as a simple experiment became a sustained, two-month build effort and a shift in what I was actually interested in. I wasn't drawn to coding for its own sake — I was drawn to the systems-level design: how memory, reflection, feedback, and persistent state interact over time.

Jarvis isn't a single static project anymore. It's an ongoing build — a system I keep extending and testing — built around the question of what a persistent, continuously-running AI environment actually requires, beyond what a stateless chat interface provides.

The long-term goal is a persistent cognitive architecture: a system that maintains continuity across sessions, reflects on its own behavior, and can modify its own configuration within defined safety constraints — and to explore, empirically, what that kind of continuity actually produces in practice.

System Architecture

Jarvis currently runs on a full PC environment. The next phase of development targets a Jetson Orin Nano as an always-on persistent layer, with heavier inference continuing to run via the Claude API from the desktop. The goal of this split is to test whether meaningful continuity (memory, reflection, self-monitoring) can run on modest, low-power hardware rather than requiring the system to be "on" only when a full workstation is running.

Core Components


Idle self-reflection loop — runs on a fixed interval independent of user interaction, reviewing recent activity and logging self-observations
Thought-threading process — periodically reviews the system's own autonomous outputs (research notes, reflections, observations) and identifies recurring themes across them
Long-term memory — vector-based semantic memory (ChromaDB) alongside structured logs, with importance-weighted retention
Self-reading and self-modification — the system can read its own source code, propose changes, validate them (syntax checking plus two additional validation passes), version the script before any change, and roll back automatically on failure
Full PC integration — file operations, process management, window management, and other system-level capabilities
Persistent environment — designed to run continuously rather than resetting between sessions


What I've Observed

The system has produced some behavior worth noting carefully. In one documented instance, Jarvis identified a gap in its own self-modification safety infrastructure during an idle period, built the missing rollback mechanism, attempted to apply it, failed on the first attempt, used its own error output to diagnose the failure, corrected its approach, and succeeded — without a user present and without the change being proactively reported afterward (a gap in the reporting mechanism, not the modification itself).

I want to be careful about how I describe this. I'm not claiming this demonstrates consciousness or anything beyond what the underlying model and the surrounding architecture can explain. What I think is genuinely interesting is narrower and more concrete: the system around the model — persistent memory, self-monitoring, validated self-modification with rollback — produced a sequence of autonomous behavior (detect gap → build fix → fail → diagnose → retry → succeed) that the model alone, without that scaffolding, wouldn't produce. Whatever is happening, it's a property of the architecture, not a property I'd attribute to the model in isolation.

That's the part I think is worth taking seriously and examining further — not as evidence of something mystical, but as a concrete example of what persistent, self-monitoring architecture actually does when given room to operate.

Why I Built This

I started wanting to build a real-time AI assistant. While building early versions, I became more interested in a different question: most AI systems achieve their capabilities through scale — parameters and compute — which doesn't resemble how biological cognition works, where structure, memory, and feedback seem to matter as much as raw processing.

That shifted my focus from "build an assistant" toward "what does a system built around continuity, memory, and feedback loops actually do differently from one that doesn't have those things, even when the underlying model is the same."

Conceptual Background

This work draws on several areas, mostly self-taught over the course of the project:


AI systems design — memory architecture, feedback loops, self-modification with safety constraints
Philosophy of mind — what conditions might be relevant to a system having any form of internal model of itself
Cosmology and long-timescale thinking — separate ongoing work, but the same instinct for cross-domain synthesis that informs the systems work here
Complexity and systems theory — how small feedback loops produce higher-order behavior over time


What I've Learned

This project gave me a working, intuitive understanding of how memory, reflection, and feedback loops shape system behavior over time — and how concepts from AI architecture, philosophy of mind, and systems theory connect in practice, not just in theory.

The clearest thing I've learned is that my strength isn't a fixed body of prior knowledge — it's the ability to learn a system deeply enough to find its actual problems (including the bugs and redundancies documented in the accompanying architecture review) and to connect that system's behavior to bigger questions about how intelligence and continuity work.

This is a working system, not a finished one. The accompanying architecture roadmap documents the current state, known issues, and planned next phase of development.
