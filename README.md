# Agentic Coding Guidelines

**Protocols and practices for cost-effective, controlled AI-assisted development.** Includes agentic coding control patterns, token efficiency strategies, code navigation through knowledge graph tool - Graphify, and session discipline for teams.

---

## Overview

This repository provides comprehensive, production-ready guidance for teams building sustainable AI-assisted development workflows. It addresses the fundamental challenges of agentic coding:

- **Runaway token costs** — How AI agents consume 60–80% of tokens on orientation
- **Lost architectural control** — How to keep human decision-making at the center
- **Context explosion** — How sessions grow expensive quadratically, not linearly
- **Team coordination** — How to standardize agent usage across a team

## What's Inside

### 📋 [Agentic Coding Control Protocol](./agentic-coding-control-protocol.md)

The core discipline framework for AI-assisted coding. Defines:

- **9 Core Principles** — YAGNI, KISS, Simplicity Over Completeness, and more
- **Naming Contracts** — Professional naming standards that prevent codebase rot
- **Simplicity Rules** — Complexity budgets, collapse checks, banned patterns
- **Code Structure Hard Limits** — Functions, classes, files, nesting constraints
- **Feature Isolation Protocol** — One feature, fully working, before the next
- **Human-in-the-Loop Question Protocol** — When agents must stop and wait
- **Review Gates** — Mandatory decision points before code moves

**Best for:** Teams implementing disciplined agent workflows, architects designing AI-assisted systems, engineering leaders establishing standards.

### 📊 [Token Efficiency Guide](./token-efficiency-guide.md)

Practical, quantified strategies to reduce token consumption by 40–70% per session.

Covers six optimization layers:

1. **Context Firewall** — `.claudeignore`, lean instruction files, path-scoped rules
2. **Static Code Intelligence** — Aider Repo Map, Graphify, CodeGraph, Repomix comparison
3. **Session Discipline** — Query order protocol, compaction strategy, subagent isolation
4. **Prompt Caching** — Anthropic & OpenAI caching mechanisms, break-even analysis
5. **Model Tiering** — Match model capability to task complexity (Haiku vs. Opus)
6. **Hooks & Enforcement** — Claude Code hooks to prevent expensive anti-patterns

**Best for:** Engineering leads optimizing budgets, DevOps teams managing agent platforms, developers concerned about API costs.

### 🔍 [Graphify — Knowledge Graph Navigation Guide](./graphify-guide.md)

Detailed guide to building and deploying **Graphify**, a knowledge graph indexing tool that enables agents to navigate large codebases without blind file reading.

Covers:

- **Quick start:** Three steps to build a queryable knowledge graph
- **How it works:** AST extraction (zero API cost), graph construction, Leiden clustering
- **Token savings:** 6.8× to 49× reduction per navigation task
- **Team setup:** What gets committed, what's per-machine, merge conflict handling
- **Platform integration:** Claude Code, Codex, Cursor, Aider, Copilot
- **Advanced usage:** MCP server, CI enforcement, cross-project graphs

**Best for:** Teams with large codebases, GenAI engineers building agent infrastructure, teams deploying Graphify.

---

## Key Concepts

### The Control Model

Agentic coding fails when the agent builds to *completion* rather than *understanding*. This framework enforces:

- **Skeleton-first review** — Signatures and data flow before implementation
- **Complexity budgets** — Every request has a 3-point budget for new classes/models/files
- **TODOs over silent handling** — Understood gaps are better than unseen edge cases
- **Human-centered decisions** — Agents propose; humans decide

### The Economics

From a cost perspective, every session has this breakdown:

```
60–80%  Orientation (finding where things are)   ← Graphify solves this
10–20%  Repeated context (re-reading files)      ← Compaction + HANDOFF.md
8–12%   System overhead (AGENTS.md, tools)       ← Context firewall
5–15%   Actual work (your prompts + answers)     ← The value you pay for
```

Graphify + session discipline + model tiering can reduce the first three categories by 40–70%.

### The Query Protocol

Every session follows this order:

1. **Read HANDOFF.md** (if it exists) — Your context, nothing else needed
2. **Query the graph** — `graphify query "what I need to know"`
3. **Targeted file read** — Only specific functions, not whole files
4. **Make the change**
5. **Run targeted tests**

This discipline prevents the agent from reading files to discover structure.

---

## Getting Started

### For Individual Developers

1. **Read** the [Agentic Coding Control Protocol](./agentic-coding-control-protocol.md) — understand the 9 principles
2. **Set up** a `.claudeignore` file (Section 4 of Token Efficiency Guide)
3. **Optimize** your AGENTS.md (Section 4 of Token Efficiency Guide)
4. **Optional:** Deploy Graphify for your project (Graphify guide, Quick Start section)

### For Engineering Teams

1. **Adopt** the control protocol as a shared standard (can be customized per team)
2. **Establish** token budgets and governance (Section 13 of Token Efficiency Guide)
3. **Deploy** Graphify to your codebases (Graphify guide, Team Setup section)
4. **Implement** hooks to enforce discipline (Section 9 of Token Efficiency Guide)
5. **Standardize** on HANDOFF.md for cross-session memory (Section 6.4 of Token Efficiency Guide)

### For Platform Builders

1. **Integrate** Graphify indexing into your platform's initialization flow
2. **Implement** pre-tool hooks to redirect agents toward graph queries
3. **Add** HANDOFF.md templating to your session-end protocol
4. **Reference** the control protocol as best practices for your agent documentation

---

## Document Structure

| Document | Audience | Length | Purpose |
|----------|----------|--------|---------|
| **Agentic Coding Control Protocol** | Architects, Tech leads, Disciplined developers | ~1000 lines | Defines what *good* agentic coding looks like; the control model |
| **Token Efficiency Guide** | Engineering leads, DevOps, Cost-conscious teams | ~1000 lines | Practical strategies to reduce costs by 40–70% |
| **Graphify Guide** | Backend engineers, Platform teams, Large codebases | ~900 lines | How to build and deploy knowledge graph navigation |

---

## Production Usage

These guidelines are battle-tested in:

- **Backend engineering teams** — Building APIs, microservices, infrastructure
- **GenAI engineering** — Building RAG systems, agent frameworks, LLM pipelines
- **Solo developers** — Freelancers and startup engineers working with Claude Code, Cursor, Aider
- **Platform teams** — Building internal AI coding platforms, agent frameworks

Real-world benchmarks from production codebases:

- **40–70% reduction** in tokens per navigation task with Graphify + session discipline
- **6.8× to 49× improvement** per query vs. blind file reading (depends on codebase size)
- **$2,000–$8,000/month savings** per engineer for teams with 10+ engineers using agents daily

---

## Core Principles at a Glance

1. **YAGNI** — Build only what the current requirement needs
2. **KISS** — Simplest structure that satisfies the requirement
3. **Breadcrumbs Over Completeness** — Understood gaps > silently handled edge cases
4. **Patterns Emerge** — Don't pre-architect enterprise patterns
5. **Decisions Belong to Humans** — Agents propose; humans approve
6. **One Feature, Fully Working** — Complete features before starting new ones
7. **Legitimacy Over Cleverness** — Follow idioms, conventions, safety patterns
8. **Context Before Code** — Discover existing code before writing new code
9. **Surface Intent** — Code contract visible without reading the body

---

## Quick Reference

### When to Use Each Document

- **Stuck in an agent session? Lost architectural control?** → Read the Control Protocol
- **Shocked by API bills? Token consumption out of control?** → Read the Token Efficiency Guide
- **Codebase too large? Agent reading files blindly?** → Read the Graphify Guide

### Integration Points

- **AGENTS.md / CLAUDE.md** — Reference the Control Protocol principles
- **Project CI/CD** — Integrate Graphify indexing (see Graphify guide, CI section)
- **.claudeignore / .aiderignore** — Use the template from Token Efficiency Guide
- **Session workflow** — Adopt the Query Protocol from Token Efficiency Guide
- **Cross-session work** — Use HANDOFF.md pattern from Token Efficiency Guide

---

## FAQ

**Q: Do I have to follow all the rules in the Control Protocol?**  
A: No. The Control Protocol is a framework. Teams should adapt it to their context. Start with the 9 principles; adjust complexity limits and named patterns based on your team's size and codebase.

**Q: Is Graphify required?**  
A: No. Graphify is the *recommended* solution for large codebases, but any code intelligence tool (Aider's repo map, CodeGraph) can work. The important thing is that agents navigate via structure, not by reading files blindly.

**Q: How do I get my team to follow these practices?**  
A: Start with the economics. Show the token breakdown. Deploy Graphify and measure savings. Implement hooks in Claude Code to enforce query discipline. Make the patterns automatic, not aspirational.

**Q: Can I use this with [platform]?**  
A: The principles are platform-agnostic. The Control Protocol applies to any agent. The Token Efficiency Guide has platform-specific sections for Claude Code, Codex, Cursor, and Aider. Graphify integrates with all major platforms.

---

## License

These guidelines are offered as-is for teams building AI-assisted development workflows. Adapt, share, and customize for your context. Attribution appreciated.

---

## Contributing

These documents are mature but not frozen. If you've found practices that work or counter-examples that break the model, please open an issue or discussion.

---

## Resources

- **Graphify** — Knowledge graph indexing tool  
  GitHub: [`graphifyy`](https://github.com/graphifyy) | PyPI: `graphifyy`

- **Aider** — AI pair programmer with repo map  
  GitHub: [paul-gauthier/aider](https://github.com/paul-gauthier/aider)

- **Claude Code** — Claude's native IDE integration  
  Docs: [Claude.ai/code](https://claude.ai)

- **Cursor** — AI-first code editor  
  Website: [cursor.com](https://cursor.com)

---

**Last Updated:** June 2026  
**Edition:** 2025–2026  
**Scope:** Claude Code · OpenAI Codex CLI · GitHub Copilot · Cursor · Aider · Gemini CLI
