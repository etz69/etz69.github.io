---
layout: "post"
title: "Cairn: Inside the Open-Source AI Framework That Finds Zero-Days on Its Own"
date: 2026-11-28 08:52:33 +0000
description: "Cairn is an open-source AI penetration-testing framework whose fact-intent graph and worker model can run autonomous recon, exploitation and post-exploitation workflows."
categories: ["security"]
tags: ["ai", "ai-agents", "security", "security-operations", "open-source", "research"]
---

# Cairn: Inside the Open-Source AI Framework That Finds Zero-Days on Its Own

**Eramind Security | 28 November 2026**

When Amazon disclosed that attackers used AI agents to autonomously discover and exploit a zero-day vulnerability in MONAI Label — an open-source medical imaging AI application — the revelation wasn’t just about the vulnerability. It was about *how* it was found. The attackers didn’t manually probe ports or craft exploits by hand. They used **Cairn**, an open-source AI penetration-testing framework, pointed it at a target IP with a single goal — “obtain shell access” — and let the agents figure out the rest. (Source: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own))

Cairn represents a shift in offensive security: from human-driven tooling to autonomous, goal-directed AI agents that plan, execute, and adapt in real time. Here’s how it works, what it can do, and what it means for defenders.

## What Cairn Is

Cairn is hosted on GitHub under the `oritera` organization ([github.com/oritera/Cairn](https://github.com/oritera/Cairn)) and released under the AGPL license. Its creators describe it not as a traditional penetration-testing toolkit with fixed modules, but as a **general-purpose AI state-space search engine** — validated first on autonomous penetration testing. (Sources: [Brandefense analysis](https://brandefense.io/odin-research/what-is/cairn); [GitHub repository](https://github.com/oritera/Cairn))

The idea is simple in concept but powerful in practice: Cairn searches for a path from a starting point (a target IP address) to a goal (such as shell access) through an unknown state space. It doesn’t come with hard-coded exploit scripts or predefined “reconnaissance” and “exploitation” modes. Instead, its agents generate their own plan at runtime, driven by the goal the operator provides and what they discover along the way.

As of late November 2026, the repository had roughly 5,500 stars, reflecting rapid interest from both security researchers and, as we’ll see, threat actors. (Source: [GitHub repository](https://github.com/oritera/Cairn))

## How Cairn Works: The Fact-Intent Graph

At the heart of Cairn is a **blackboard architecture** — a form of stigmergic coordination where agents don’t talk to each other directly. Instead, they coordinate through a shared graph that acts as a communal workspace. The graph is built from three primitives:

| Primitive | What It Represents | Example |
|-----------|-------------------|---------|
| **Fact** | A confirmed, objective finding | Port 8080 is open; login credentials work |
| **Intent** | A planned direction of exploration, not yet executed | Test the `/api/v1/upload` endpoint for path traversal |
| **Hint** | Human judgment injected at any time | “Focus on the medical imaging endpoints” |

The graph grows from an `origin` node toward a `goal` node. Every new Fact becomes a stepping stone; every Intent is a step into the unknown. Agents observe the full graph, orient themselves to the current state, decide on the next Intents, act on them, and write the results back as new Facts — an OODA loop (Observe–Orient–Decide–Act) running continuously until the goal is met or the operator stops it. (Sources: [Brandefense](https://brandefense.io/odin-research/what-is/cairn); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## The Three Components

Cairn’s deployment splits into three parts:

1. **Cairn Server** — maintains graph consistency. It is the single source of truth for the fact-intent graph but does not execute actions itself.
2. **Cairn Dispatcher** — the sole scheduler and writer. It reads the graph, picks Intents ready to execute, spins up worker containers, feeds them prompts plus graph context, collects structured output, and writes Facts back.
3. **Worker Containers** — each project gets its own container, inside which multiple Agent Workers run concurrently. Workers can also run in local mode (without Docker) directly on the dispatcher host.

Workers receive a prompt and return **structured output** — not free-form chat. They never communicate directly with each other; all coordination happens through the shared graph. (Sources: [Brandefense](https://brandefense.io/odin-research/what-is/cairn); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## Agent Strategies: Goal-Driven, Not Mode-Driven

Cairn does not ship with separate agents for “reconnaissance,” “vulnerability scanning,” or “exploitation.” Instead, the same generic worker pool performs whatever phase is needed to reach the goal:

- **Reconnaissance** emerges when agents generate Intents to enumerate services, scan ports, and map the attack surface.
- **Vulnerability discovery** emerges when agents propose Intents to test parameters, probe endpoints, or try known exploit patterns.
- **Exploitation** emerges when a Fact reveals a viable path — agents craft and execute payloads, writing results back as new Facts.
- **Post-exploitation** can include activities such as establishing persistent access or moving within the network. Observed techniques include writing reverse shells to establish command-and-control channels, modifying `~/.ssh/authorized_keys` to persist SSH access, and privilege escalation through misconfigured services — each step recorded as another Fact/Intent cycle. (Sources: [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks); [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own))

The operator supplies one high-level goal (e.g., “obtain shell access on target X”), and Cairn’s orchestration loop decomposes it into concrete actions at runtime. The graph *is* the plan. (Sources: [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks); [Google Threat Intelligence Group](https://cloud.google.com/blog/topics/threat-intelligence/agentic-threat-activity-crosses-a-significant-threshold))

## The Orchestration Loop: Reason → Explore

From the AWS Security Blog and Unit 42 reconstructions of the MONAI Label incident, Cairn’s loop works in two phases:

1. **Reason Phase** — the AI model reviews everything found so far (the entire fact-intent graph) and decides what to try next, generating new Intents.
2. **Explore Phase** — the model carries out one action through a worker, and the framework records the result as a new confirmed Fact.

The loop repeats until the model judges the goal met, options exhausted, or the operator intervenes. In observed operations, individual runs lasted from 20 minutes to more than 50 hours with no attacker-imposed time limit. (Sources: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## Real-World Use: The Amazon MONAI Label Incident

In September 2026, Amazon identified a publicly exposed Cairn server operated by a China-linked threat actor. Because the server lacked authentication, Amazon could observe the operators’ prompts and outputs in real time — written in Simplified Chinese, with timestamps matching China Standard Time. (Source: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own))

The attackers used Cairn paired with Zhipu GLM and DeepSeek models. They gave the agents a list of IP addresses and one goal: “obtain shell access.” The agents autonomously discovered and exploited a path-traversal zero-day in MONAI Label running on a target server, gaining full control of the system with little human input. Amazon assessed the flaw as a zero-day with high confidence — no fix, no advisory, and no prior vulnerability-database mention at the time. (Sources: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own); [Ars Technica](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input))

## Security Implications

Cairn lowers the barrier for autonomous offensive operations. A single operator can launch a campaign and let AI agents handle reconnaissance, exploitation, and post-exploitation without constant oversight. For defenders, this means:

- **Attacks can run unattended for hours or days**, increasing the window of exposure.
- **Traditional signature-based detection may miss novel exploit chains** that agents assemble on the fly.
- **Unauthenticated, publicly exposed management interfaces** (like Cairn’s dispatcher) become critical risk points — as they did in the Amazon case.

The same capabilities that help security researchers test defenses can be turned against them. Cairn is open-source, AGPL-licensed, and publicly available — which means its power is accessible to both sides.

## Conclusion

Cairn is not just another pentesting tool. It’s a glimpse into a future where offensive security is increasingly autonomous, goal-directed, and AI-driven. Understanding how it works — its blackboard architecture, fact-intent graph, and goal-driven planning — is essential for anyone responsible for defending systems in this new era.

## Sources

- GitHub: github.com/oritera/Cairn
- Brandefense analysis: brandefense.io/odin-research/what-is/cairn
- AWS Security Blog: aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own
- Ars Technica: arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input
- Unit 42 / Palo Alto Networks: unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks
- Google Threat Intelligence Group: cloud.google.com/blog/topics/threat-intelligence/agentic-threat-activity-crosses-a-significant-threshold
