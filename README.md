# SALSA 💃

**Enterprise Secure AI-Native Operating System for Agile at Scale. Powered by fast-asdlc.**

**Reduce Enterprise TTM by 5x with a secure, AI-native Agentic Development Framework.**  
*Open-source | Open-Model Native | Human-in-the-loop | Architecturally Rigorous*

---

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![License: MIT](https://shields.io)](https://opensource.org)
[![GitHub stars](https://shields.io)](https://github.com)
[![GitHub forks](https://shields.io)](https://github.com)

---

## 🎯 Overview

**SALSA (Scaled Agentic Lean Secure Agile)** is an executable, AI-native operating system designed to automate and orchestrate enterprise-scale agile workflows. By upgrading traditional, document-heavy agile methodologies into an automated ecosystem of coordinated AI agents, SALSA slashes Time-to-Market (TTM) while maintaining strict architectural determinism and enterprise-grade data security.

The core engine leverages **fast-asdlc** concepts to treat methodology, rules, and workflows as code (`Everything-as-Code`). It eliminates chat-bot hallucinations by compiling declarative role runtimes, enforcing strict input/output contracts, and executing automated Quality Gates.

## 🛡️ Core Pillars

* **Enterprise-Grade Security:** Designed for `Local-LLM first` deployments (e.g., Qwen-2.5-Coder, GLM). Your strategic roadmaps, codebases, and business backlogs never leave your secure corporate infrastructure.
* **Spec-Driven & Deterministic:** No agent acts without clear boundaries. Every role runs within a compiled runtime environment with hard guardrails defined by expert Agile Coaches and SPCs.
* **Context Efficiency:** Heavily optimized for Mermaid.js diagrams and structured Markdown to compress context windows, radically cutting down LLM token costs and processing time.
* **Human-in-the-Loop (HITL):** Seamless integration of approval gates. Autonomous agents generate artifacts directly in Git via Pull Requests, which require explicit human sign-off before merging.

## 📂 Architecture & Directory Structure

```text
.agents/
├── fast-asdlc/             # Underlying core engineering engine
│   └── BOOTSTRAP-PROMPT.md # Bootstrapping prompt for agent generation
├── salsa-core/             # SALSA Methodology & Master Guardrails Layer
│   └── methodology.md      # Scaled Agentic Lean Secure Agile rules & patterns
└── salsa-factories/         # Compiled Agent Runtimes & Factories
    └── roles/              # Enterprise Agile Digital Twins (15+ roles)
        ├── epic-owner/     # System prompts, MCP tools, and Quality Gates
        ├── rte/            # Release Train Engineer coordination automation
        └── product-mgmt/   # Feature breakdown and WSJF metrics engine
```

## 🚀 Getting Started (Workshop Quickstart)

### Prerequisites
* Cloned repository inside your secure development environment.
* Configured local or private LLM endpoints (recommended: Qwen 2.5 Coder 72B).
* Installed Model Context Protocol (MCP) server for enterprise integrations (Jira, Git, Confluence).

### Bootstrap the Project Manager Agent
To initialize the AI Project Manager for your implementation stream using the self-bootstrapping engine:

```bash
# 1. Add your profile to the team roster
cp .members/member_template.md .members/your_name.md
# 2. Fill in your expertise tags and time commitment budget
# 3. Trigger the Meta-Agent bootstrap routine
fast-asdlc init --config=.agents/salsa-core/methodology.md
```

## ⚖️ Legal Disclaimer

*SALSA is an independent, open-source project built on top of the fast-asdlc engine. SAFe® and Scaled Agile Framework® are registered trademarks of Scaled Agile, Inc. This project is not affiliated with, endorsed, or sponsored by Scaled Agile, Inc. It serves as an autonomous, technological implementation layer developed independently by the agile community.*
