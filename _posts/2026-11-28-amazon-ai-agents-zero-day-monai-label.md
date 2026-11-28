---
layout: "post"
title: "Amazon says AI agents found a zero-day on their own"
date: 2026-11-28 07:27:46 +0000
description: "Amazon says attackers used AI agents to find and exploit a zero-day in MONAI Label with little human input."
categories: ["security"]
tags: ["ai", "ai-agents", "security", "open-source", "security-operations", "llm"]
---

Amazon reported a case in which attackers used AI agents to find and exploit a zero-day vulnerability in MONAI Label, an open-source medical imaging AI application, and gain full control of a target system. The report matters because it shows a real attack chain with little human guidance: the operators gave the agents an IP list and an objective, and the agents did the rest [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own), [Ars Technica](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input).

## What happened
Amazon said the vulnerability was a path-traversal zero-day in MONAI Label running on a server in the target environment. The company assessed it as a zero-day with high confidence because, at the time, there was no fix, no advisory and no prior vulnerability-database entry. Amazon also said the attackers used Cairn, an open-source AI penetration-testing framework, and gave the agents two inputs: a list of IP addresses and the goal to obtain shell access [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own), [Ars Technica](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input).

## How the attack worked
According to Amazon, the AI agents discovered the weakness on their own, then exploited it without much human intervention. That meant the operators were not acting as manual exploit writers for every step. They provided a target list and a goal, and the agents generated the scanning, discovery and exploitation path needed to reach a shell [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own).

Amazon said it first identified a publicly exposed Cairn server on 10 September 2026. Because the server had no authentication, Amazon could observe project settings, prompts and outputs. The prompts were written in Simplified Chinese, and the timestamps matched China Standard Time. Amazon described the actor as Chinese-nexus or China-linked, while the confidence level varied by claim [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own).

## What Amazon disclosed
The report is careful in one important way: it separates what was observed from what was inferred. Amazon described the activity, the tooling and the language used in the prompts. It also stated where its confidence was high and where it was lower. That distinction matters, because attribution and technique are not the same thing [AWS Security Blog](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own).

## Why it matters
This incident is not just about one bug in one application. It shows that autonomous agents can lower the cost of finding and using a vulnerability, especially when they are given a clear target and a clear goal. For defenders, that raises the value of exposure reduction, patch hygiene and monitoring for unusual agent-like behavior around scanning, exploitation and shell access.

It also shows why security teams need to treat agentic tooling as operationally real. A framework built for testing can be repurposed for attack work. When that happens, the speed of discovery can exceed the speed of human review.

## Sources
- [AWS Security Blog: Chinese-nexus actor’s Cairn AI agents found a zero-day on their own](https://aws.amazon.com/blogs/security/chinese-nexus-actors-cairn-ai-agents-found-a-zero-day-on-their-own), 20 November 2026
- [Ars Technica: Amazon says hackers’ AI agents found and exploited 0-day with little human input](https://arstechnica.com/security/2026/11/amazon-says-hackers-ai-agents-found-and-exploited-0-day-with-little-human-input), 27 November 2026
