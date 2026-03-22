# axis-workflow-demo

Policy-governed AI workflow engine — AML/CTF compliance, AI agent governance, and insider threat detection. Patent pending.

---

## What This Is

This repository contains demonstration specifications and annotated sample output for **ax-workflow** — a governance engine built on a single foundational principle:

> Every input is either completely verified or explicitly handled as unknown. There is no third option. And the entire execution history is deterministically replayable.

This is not a logging tool. Not an audit wrapper. Not a compliance dashboard bolted onto an existing system.

It is a governance architecture where the regulatory obligations are enforced by construction — not by policy manuals, not by training, not by supervision. By architecture.

The engine implementation is proprietary. This repository contains specification files and sample ledger output that demonstrate the architecture is real, coherent, and regulatorily defensible.

---

## The Five Architectural Guarantees

### 1. AI is non-authoritative by construction
AI appears in this system in exactly two roles: assessment and narrative drafting. In both cases it produces a structured artifact. It never routes a workflow. It never makes a decision. It never submits a regulatory report. This is verifiable by reading the route files — a non-technical person can confirm it.

### 2. Every input is either completely verified or explicitly handled as unknown
There is no middle ground. A novel fraud pattern, a new laundering typology, an agent instruction outside defined bounds — none of these slip through. They hit the unknown handler, which is itself a governed, auditable, policy-defined execution path. The gap that fraud exploits collapses to zero by construction.

### 3. Humans are part of the state machine, not side channels
`(wait-for human-decision)` makes every human decision auditable, pausable, and replayable. The ledger records when the decision arrived, who made it, what they said, and what the policy engine did with it. Humans cannot act outside the defined escalation paths.

### 4. Policy is the only gatekeeper — for everyone
Every decision — AI output, human input, external signal — passes through the policy gate. No authority level bypasses it. No actor has special privileges. The policy engine is the only authority.

### 5. The entire history is deterministically replayable
The append-only ledger records complete input state at every decision point. Any historical run can be re-executed exactly — not reconstructed from logs, but deterministically re-run. And before any policy change is deployed, blast radius analysis replays the complete execution history against the proposed change, producing an exact diff of every decision that would have changed.

---

## Two Additional Capabilities

### Complete Replay
Take any historical run. Restore it from the ledger. Re-execute it exactly as it ran — 14 months ago, under the exact conditions that existed at the time. For regulatory audit, for dispute resolution, for proving a decision was correct at the time it was made.

### Blast Radius Analysis
Propose a policy change. Replay the last six months of real executions against it. Get an exact diff — which decisions would have changed, which cases would have been newly escalated, which reports would have been filed that weren't. Before a single line hits production. Not a simulation. Every real execution. Exact.

---

## Jurisdiction Demos

The same engine runs across every jurisdiction. The policy definitions and route files change. The architectural guarantees do not.

| Jurisdiction | Regulator | Obligation | Status |
|---|---|---|---|
| 🇦🇺 Australia | AUSTRAC | Suspicious Matter Report (SMR) | [Available](./au/) |
| 🇺🇸 United States | FinCEN | Suspicious Activity Report (SAR) | Coming soon |
| 🇪🇺 European Union | EBA / National FIUs | Suspicious Transaction Report (STR) | Coming soon |

---

## What Changes Between Jurisdictions

| Component | Changes | Stays the Same |
|---|---|---|
| Route definition | Jurisdiction-specific states and timers | Route structure and invariants |
| Policy rules | Regulatory thresholds and obligations | Policy gate architecture |
| AI contexts | Domain-specific assessment prompts | AI non-authoritative guarantee |
| Regulatory timers | Jurisdiction-specific deadlines | Timer gate enforcement mechanism |
| Compliance claims | Mapped to local regulation | Unknown-as-first-class guarantee |

**The engine is jurisdiction-agnostic.** Swap the policy and route definitions. Every architectural guarantee holds.

---

## What This Repository Contains

```
axis-workflow-demo/
├── README.md                        ← This file
├── au/                              ← Australia — AUSTRAC SMR
│   ├── README.md                    ← Annotated walkthrough, four paths
├── us/                              ← United States — FinCEN SAR (coming)
└── eu/                              ← European Union — AMLD6 STR (coming)
```

---

## Who This Is For

**Compliance directors and risk officers** — the `au/README.md` walkthrough is written for you. No technical background required. An auditor can read the route file. That is rare.

**Security architects** — the policy gate architecture, unknown-as-first-class-state primitive, and decomposable chain of custody are described in full in the specification files.

**Regulators** — every regulatory obligation is mapped explicitly to an architectural property. The system cannot be made to violate the obligation without changing the architecture.

**Enterprise technology teams** — the engine sits on top of existing systems. No rip and replace. No stranded investment. Incremental deployment from a single process outward.

---

## Applying This to AI Agent Networks

The same engine governs networks of AI agents. Every agent action passes through the same policy gate. Every agent output is either within defined bounds or explicitly handled as unknown. The decomposable architecture means the same governance applies recursively to every sub-agent and subprocess — unbroken chain of custody from the highest-level orchestrator to the lowest-level data capture.

Before any agent behaviour change is deployed, blast radius analysis shows exactly which decisions across your entire agent network would have changed across every historical run.

Provably safe. Auditable. Change-managed. By construction.

---

## Patent Status

The following inventions are protected by provisional patent applications filed with IP Australia:

- **Honeypot State with Dual Ledger Streams** — AU provisional filing 2026901859
- **Blast Radius Analysis via Deterministic Ledger Replay** — AU provisional filing 2026901857
- **Unknown as First-Class Execution State, AI Agent Network Governance, and Semantic Snapshot Testing** — AU provisional filing 2026902500

*Patent pending. All rights reserved.*

---

## Contact

Enquiries regarding licensing, pilot engagements, and commercial discussions:

**Christopher John Taylor**
chris.taylor.dev@proton.me
linkedin.com/in/chris-taylor-3494852

---

© 2026 Christopher John Taylor. All rights reserved. Patent pending.