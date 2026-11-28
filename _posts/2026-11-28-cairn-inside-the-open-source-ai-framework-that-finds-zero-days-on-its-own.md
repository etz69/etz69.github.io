---
layout: "post"
title: "Cairn: Inside the Open-Source AI Framework That Finds Zero-Days on Its Own"
date: 2026-11-28 08:52:33 +0000
description: "Cairn is an open-source AI penetration-testing framework whose fact-intent graph and worker model can run autonomous recon, exploitation and post-exploitation workflows."
categories: ["security"]
tags: ["ai", "ai-agents", "security", "security-operations", "open-source", "research"]
---

# Cairn: Inside the Open-Source AI Framework That Finds Zero-Days on Its Own

Cairn is an open-source AI penetration-testing framework whose fact-intent graph and worker model can run autonomous recon, exploitation, and post-exploitation workflows.

## What Cairn Is

Cairn is hosted on GitHub under the `oritera` organization ([github.com/oritera/Cairn](https://github.com/oritera/Cairn)) and released under the AGPL license. Its creators describe it not as a traditional penetration-testing toolkit with fixed modules, but as a general-purpose AI state-space search framework.

The idea is simple in concept but powerful in practice: Cairn searches for a path from a starting point (a target IP address) to a goal (such as shell access) through an unknown state space. It doesn’t come with hard-coded exploit scripts or predefined reconnaissance and exploitation modes. Instead, it lets agents reason over a shared graph of facts and intents and decide what to do next.

As of late November 2026, the repository had roughly 5,500 stars, reflecting rapid interest from both security researchers and, as we’ll see, threat actors. (Source: [GitHub repository](https://github.com/oritera/Cairn))

## How Cairn Works: The Fact-Intent Graph

Cairn’s core abstraction is a blackboard-style graph of two node types:

| Node type | What it means | Example |
| --- | --- | --- |
| **Fact** | A confirmed observation or result | “Port 443 is open” |
| **Intent** | A proposed action or hypothesis | “Try SQL injection on login endpoint” |

Agents write Intents into the graph, workers execute them, and the results come back as new Facts — an OODA loop (Observe–Orient–Decide–Act) running continuously until the goal is met or the operator stops it. (Sources: [Brandefense](https://brandefense.io/odin-research/what-is/cairn); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## The Three Components

Cairn’s architecture has three moving parts:

1. **Dispatcher** — coordinates the overall project and manages the graph.
2. **Reasoner** — the model that turns the current graph into the next Intents.
3. **Worker Containers** — each project gets its own container, inside which multiple Agent Workers run concurrently. Workers can also run in local mode (without Docker) directly on the dispatcher host.

These components are designed so the agent can reason, act, and record results without free-form chat. They never communicate directly with each other; all coordination happens through the shared graph. (Sources: [Brandefense](https://brandefense.io/odin-research/what-is/cairn); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## Agent Strategies: Goal-Driven, Not Mode-Driven

Cairn does not ship with separate agents for reconnaissance, vulnerability scanning, or exploitation. Instead, the same generic worker pool performs whatever phase is needed to reach the goal:

- **Reconnaissance** emerges when agents generate Intents to enumerate services, scan ports, and map the attack surface.
- **Vulnerability discovery** emerges when agents propose Intents to test parameters, probe endpoints, or try known exploit patterns.
- **Exploitation** emerges when a Fact reveals a viable path — agents craft and execute payloads, writing results back as new Facts.
- **Post-exploitation** can include activities such as establishing persistent access or moving within the network. Observed techniques include writing reverse shells to establish command-and-control channels, modifying `~/.ssh/authorized_keys` to persist SSH access, and privilege escalation through misconfigured services — each step recorded as another Fact/Intent cycle. (Sources: [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks); [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own))

The operator supplies one high-level goal, and Cairn’s orchestration loop decomposes it into concrete actions at runtime. The graph is the plan. (Sources: [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks); [Google Threat Intelligence Group](https://cloud.google.com/blog/topics/threat-intelligence/agentic-threat-activity-crosses-a-significant-threshold))

## The Orchestration Loop: Reason → Explore

From the AWS Security Blog and Unit 42 reconstructions of the MONAI Label incident, Cairn’s loop works in two phases:

1. **Reason Phase** — the AI model reviews everything found so far and decides what to try next, generating new Intents.
2. **Explore Phase** — the model carries out one action through a worker, and the framework records the result as a new confirmed Fact.

The loop repeats until the model judges the goal met, options exhausted, or the operator intervenes. In observed operations, individual runs lasted from 20 minutes to more than 50 hours with no attacker-imposed time limit. (Sources: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own); [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks))

## Real-World Use: The Amazon MONAI Label Incident

In September 2026, Amazon identified a publicly exposed Cairn server operated by a China-linked threat actor. Because the server lacked authentication, Amazon could observe the operators’ prompts and outputs in real time — written in Simplified Chinese, with timestamps matching China Standard Time. (Source: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own))

The attackers used Cairn paired with Zhipu GLM and DeepSeek models. They gave the agents a list of IP addresses and one goal: “obtain shell access.” The agents autonomously discovered and exploited a path-traversal zero-day in MONAI Label running on a target server, gaining full control of the system with little human input. Amazon assessed the flaw as a zero-day with high confidence — no fix, no advisory, and no prior vulnerability-database mention at the time. (Sources: [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own); [Ars Technica](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input))

## Security Implications

Cairn lowers the barrier for autonomous offensive operations. A single operator can launch a campaign and let AI agents handle reconnaissance, exploitation, and post-exploitation without constant oversight. For defenders, this means:

- Attacks can run unattended for hours or days, increasing the window of exposure.
- Traditional signature-based detection may miss novel exploit chains that agents assemble on the fly.
- Unauthenticated, publicly exposed management interfaces — like Cairn’s dispatcher — become critical risk points, as they did in the Amazon case.

The same capabilities that help security researchers test defenses can be turned against them. Cairn is open-source, AGPL-licensed, and publicly available — which means its power is accessible to both sides.

## Conclusion

Cairn is not just another pentesting tool. It’s a glimpse into a future where offensive security is increasingly autonomous, goal-directed, and AI-driven. Understanding how it works — its blackboard architecture, fact-intent graph, and goal-driven planning — is essential for anyone responsible for defending systems in this new era.

## Sources

- [GitHub: github.com/oritera/Cairn](https://github.com/oritera/Cairn)
- [Brandefense analysis](https://brandefense.io/odin-research/what-is/cairn)
- [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own)
- [Ars Technica](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input)
- [Unit 42 / Palo Alto Networks](https://unit42.paloaltonetworks.com/cairn-framework-ai-autonomous-attacks)
- [Google Threat Intelligence Group](https://cloud.google.com/blog/topics/threat-intelligence/agentic-threat-activity-crosses-a-significant-threshold)
