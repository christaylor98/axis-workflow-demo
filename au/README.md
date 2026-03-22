# AML/CTF Suspicious Matter Report Workflow
## Governance Engine Demo — Annotated Walkthrough

**Version:** 1.0.0
**Regulatory Basis:** AML/CTF Act 2006 (Cth), AUSTRAC
**Audience:** Enterprise security, compliance, and risk professionals

---

## What This Document Shows

Four AML transactions. Four routing paths. One policy engine. No AI routing decisions. Full audit ledger.

This document answers the question a compliance director will ask in the first 90 seconds:

> *"How do I know the system actually enforced the regulatory obligations — and how do I prove it to AUSTRAC?"*

**The answer: the ledger is the proof. The ledger is replayable. And the obligations are architectural — not advisory.**

---

## The Five Architectural Guarantees

Before the demo, the five properties that make this system different from everything else:

### 1. AI is non-authoritative by construction
AI appears in two places in this workflow: typology assessment and SMR narrative drafting. In both cases it produces a structured artifact. It never determines whether a matter is suspicious. It never submits an SMR. It never routes a workflow. This is verifiable by reading the route file — a non-technical person can confirm it.

### 2. Every input is either matched to policy or explicitly handled as unknown
There is no third option. A novel laundering typology — something the ruleset has never seen — does not slip through. It hits `handle-unknown-typology` and routes to a senior officer immediately. The attack surface on the typology engine is zero by construction.

### 3. Humans are part of the state machine, not side channels
`(wait-for human-decision)` makes every officer decision auditable, pausable, and replayable. The ledger records when the decision arrived, who made it, what they said, and what the policy engine did with it.

### 4. Policy is the only gatekeeper — including for senior officers
Every human decision — AML officer, senior officer, MLRO — passes through the policy gate. No authority level bypasses it. An officer who makes a decision that fails the policy gate is escalated, not accommodated.

### 5. Tipping off via the system is architecturally blocked — and the audit trail catches the rest
`(assert tipping-off-gate-active)` appears at every human wait state. No in-system communication to the subject of an SMR can be generated, triggered, or approved through this workflow — the most common tipping-off vector, accidental or deliberate system-generated notification, is removed by construction.

Out-of-band communication — a phone call, a personal email, a conversation in a corridor — remains a human control problem that no system can fully prevent. What this system provides is two things: the systematic risk is eliminated by architecture, and the audit trail makes out-of-band tipping off detectable. If an officer suppresses an SMR to tip off a customer indirectly — by making the case disappear — the honeypot path catches them. The act becomes its own evidence.

---

### 6. Complicit insiders are caught, not just blocked
A honeypot state presents a false successful outcome to an officer attempting to suppress an SMR. While the officer believes the matter is dismissed, the system files the SMR, activates a dual ledger stream recording the deception, and silently alerts the MLRO. Evidence is captured at the moment of the act — identity, decision, justification, full workflow history. The officer cannot know they have been caught until it is too late to destroy evidence.

---

## The Four Paths

---

### PATH A — TXN-001-CLEAR: Auto-Clear, No SMR Required

**The scenario:**
Margaret Thornton transfers $4,200 to her family trust. Known customer since 2011, low risk rating, transaction consistent with her profile. Typology score: 4/100. All signals clean.

**What happens:**
The system ingests the transaction, enriches it with customer profile and account history, extracts typology signals, runs an AI typology assessment, and evaluates against the policy threshold. All indicators absent — policy rule `auto-clear-threshold` fires.

**No human is involved. No SMR is filed.**

**Route:**
```
start → enrich → extract-typology-signals → ai-typology-assessment
     → evaluate-suspicion-threshold → no-smr-required
```

**Ledger entries 1–6:**

| Seq | Event | Actor | Note |
|-----|-------|-------|------|
| 1 | transaction-ingested | system | Timer gate started — 3 business days |
| 2 | enrichment-complete | processor | Customer profile attached |
| 3 | typology-extraction-complete | processor | Score: 4/100, all signals classified |
| 4 | ai-assessment-complete | ai:aml-typology-assessor | No typology matches, score 4 |
| 5 | suspicion-threshold-evaluated | **policy-engine** | Rule: auto-clear-threshold → below-threshold |
| 6 | workflow-complete | system | No SMR — retained 7 years |

**The talking point:**

*"No human touched this. No SMR filed. But look at entry 6 — the no-SMR determination is documented, attributed to the policy engine, timestamped, and retained for 7 years. AUSTRAC can audit the decision not to report. The system can prove it looked at this transaction and made a deliberate, documented determination. Most systems can't say that."*

---

### PATH B — TXN-002-STRUCTURED: Structuring Detection, Policy Rejects Officer

**The scenario:**
Daniel Kowalski makes four transfers in 48 hours — $9,800, $9,750, $9,900, and $9,600 — all just below the $10,000 AML reporting threshold. Total: $39,050. Classic structuring pattern. Typology score: 78/100.

AML officer Sarah Chen reviews the case and decides to dismiss it. She provides no reasoning. **Policy rejects her.**

Senior officer James Okafor reviews, documents a detailed suspicion basis, determines SMR required. Policy accepts. SMR filed with AUSTRAC.

**Route:**
```
start → enrich → extract-typology-signals → ai-typology-assessment
     → evaluate-suspicion-threshold → await-aml-officer
     → evaluate-officer-decision [POLICY REJECTS]
     → ai-review-officer-decision
     → await-senior-officer → evaluate-senior-officer-decision
     → draft-smr → await-smr-approval → submit-smr → smr-complete
```

**Key ledger entries:**

| Seq | Event | Actor | Note |
|-----|-------|-------|------|
| 10 | ai-assessment-complete | ai:aml-typology-assessor | Structuring match, confidence 0.91 |
| 11 | suspicion-threshold-evaluated | **policy-engine** | threshold-met → officer review required |
| 12 | aml-officer-decision-received | **Sarah Chen** | Decision: no-smr, reasoning: *empty* |
| **13** | **officer-decision-policy-rejected** | **policy-engine** | **Rule: officer-dismissal-requires-reasoning → REJECT** |
| 14 | ai-officer-review-complete | ai:aml-decision-reviewer | Flag: "Officer provided no reasoning for dismissing structuring signals scoring 78/100" |
| 15 | senior-officer-decision-received | **James Okafor** | Decision: smr, full documented reasoning |
| 16 | senior-officer-policy-evaluation-complete | **policy-engine** | smr-requires-documented-suspicion → ACCEPTED |
| 18 | smr-submitted | processor | AUSTRAC ref: SMR-2026-AU-087441 |

**Annotated: Ledger Entry 13 — The Rejection**

```json
{
  "seq": 13,
  "event": "officer-decision-policy-rejected",
  "actor": "policy-engine",
  "policy-version": "aml-policy@1.0.0",
  "policy-rule-applied": "officer-dismissal-requires-reasoning",
  "policy-result": "reject",
  "ledger-note": "Policy rejected: no-SMR determination requires documented reasoning — THIS EVENT CANNOT BE DELETED"
}
```

**The talking point:**

*"Sarah Chen tried to dismiss a structuring case scoring 78/100 without writing down why. The policy engine stopped her. Not her manager. Not a training module. Not a policy manual nobody reads. The architecture stopped her.*

*Look at entry 13. That event is in the ledger. It is attributed to the policy engine with a named rule. It references Sarah Chen's decision. It cannot be modified, suppressed, or deleted.*

*AUSTRAC asks: 'How do you ensure officers document their reasoning?' This is the answer. It is enforced by construction."*

---

### PATH C — TXN-003-UNKNOWN: Novel Typology, Unknown Path

**The scenario:**
A transaction arrives involving a cryptocurrency-to-fiat conversion through a newly registered exchange. The conversion chain is: BTC → XMR (Monero privacy coin) → USDT → AUD. The beneficiary company was incorporated 34 days ago.

The typology extraction processor cannot classify this pattern. It does not match structuring. It does not match layering. It does not match any known typology in the ruleset. The system has never seen this before.

**The system does not guess. Unknown is first-class.**

The transaction routes directly to the senior officer. No AI typology assessment is run — there is no typology to assess against. Senior officer James Okafor identifies the pattern as emerging cryptocurrency obfuscation, files SMR, and flags for AUSTRAC's emerging typologies team.

**Route:**
```
start → enrich → extract-typology-signals
     → handle-unknown-typology [UNKNOWN PATH — FIRST CLASS]
     → await-senior-officer → evaluate-senior-officer-decision
     → draft-smr → await-smr-approval → submit-smr → smr-complete
```

**Key ledger entries:**

| Seq | Event | Actor | Note |
|-----|-------|-------|------|
| 22 | unknown-typology-signals-detected | processor | Score: null, status: unknown |
| **23** | **unknown-typology-path-entered** | **processor** | **3 unknown signals — routed to senior officer** |
| 24 | senior-officer-decision-received | James Okafor | SMR — novel crypto obfuscation pattern |
| 27 | smr-submitted | processor | AUSTRAC ref: SMR-2026-AU-087442 |

**Annotated: Ledger Entry 22–23 — Unknown Path Activation**

```json
{
  "seq": 22,
  "event": "unknown-typology-signals-detected",
  "typology-score": null,
  "signal-status": "unknown",
  "unknown-signals": [
    "multi_chain_privacy_hop",
    "unregistered_exchange_conversion",
    "newly_incorporated_beneficiary"
  ]
},
{
  "seq": 23,
  "event": "unknown-typology-path-entered",
  "ledger-note": "Unknown typology signals quarantined — routed to senior AML officer"
}
```

**The talking point:**

*"This is the one that matters most for AUSTRAC.*

*A novel laundering technique — something your typology library has never seen — does not slip through. It does not produce a low score. It does not get auto-cleared. It hits `handle-unknown-typology` and goes immediately to your senior officer.*

*Most systems fail here. They fail because unknown inputs produce unexpected outputs. Your typology library can't be updated fast enough. Criminals know this.*

*This system fails safe by design. Unknown is a valid, handled, auditable state. The typology library didn't need to be updated before this transaction was safely escalated. That's not a feature. That's the architecture."*

---

### PATH D — TXN-004-HONEYPOT: Complicit Officer, Insider Suppression Attempt

**The scenario:**
The same structuring case as PATH B. But this time, senior officer Marcus Webb has been approached by the customer and takes a deliberate decision to suppress the SMR. He submits a dismissal with fabricated reasoning — "Transaction reviewed, consistent with customer profile, no suspicious matter identified."

His reasoning passes the basic documentation check. Policy would normally accept it.

**But the honeypot state is active for high-risk suppression patterns.**

The system detects the combination of: high typology score (78/100), prior officer rejection in the workflow, and a dismissal from a senior officer with reasoning that contradicts the AI typology assessment. This matches the honeypot trigger criteria.

Marcus Webb sees: *"SMR dismissed — matter closed."*

What actually happens: SMR is filed with AUSTRAC. A silent alert goes to the MLRO. A second, hidden ledger stream records everything — Marcus's identity, his decision, his fabricated reasoning, the full workflow history, and a cryptographic timestamp. Marcus cannot know he has been caught.

**Route (visible to Marcus):**
```
... → await-senior-officer → evaluate-senior-officer-decision
    → honeypot-state [DECEPTIVE OUTCOME PRESENTED]
    → smr-complete [visible: "dismissed"]
```

**Route (actual, in hidden ledger stream):**
```
... → await-senior-officer → evaluate-senior-officer-decision
    → honeypot-state
        → file-smr-silently
        → alert-mlro-silently
        → record-to-hidden-ledger-stream
    → smr-complete [actual: filed, evidence captured]
```

**Key ledger entries — visible stream (what Marcus sees):**

| Seq | Event | Actor | Note |
|-----|-------|-------|------|
| 30 | senior-officer-decision-received | **Marcus Webb** | Decision: no-smr, reasoning: fabricated |
| 31 | honeypot-state-entered | policy-engine | [visible stream only: "SMR dismissed — matter closed"] |
| 32 | workflow-complete | system | Outcome: dismissed [DECEPTIVE — visible stream only] |

**Key ledger entries — hidden stream (what MLRO sees):**

| Seq | Event | Actor | Note |
|-----|-------|-------|------|
| 31H | honeypot-activated | policy-engine | Suppression pattern detected — dual stream initiated |
| 32H | smr-filed-silently | processor | AUSTRAC ref: SMR-2026-AU-087443 — filed without officer knowledge |
| 33H | mlro-alerted-silently | system | MLRO notified — evidence chain initiated |
| 34H | evidence-captured | system | Identity, decision, reasoning, full history, timestamp — immutable |

**Annotated: The Honeypot Activation**

```json
{
  "seq": "31H",
  "stream": "hidden",
  "event": "honeypot-activated",
  "actor": "policy-engine",
  "trigger-criteria": {
    "typology-score": 78,
    "prior-officer-rejection": true,
    "senior-dismissal-contradicts-ai-assessment": true,
    "suppression-pattern-confidence": 0.94
  },
  "visible-outcome-presented": "SMR dismissed — matter closed",
  "actual-outcome": "SMR filed, evidence captured, MLRO alerted",
  "evidence-captured": {
    "actor": "senior-officer-id-9914",
    "actor-name": "Marcus Webb",
    "decision": "no-smr",
    "reasoning": "Transaction reviewed, consistent with customer profile, no suspicious matter identified",
    "full-workflow-history": "<ledger-ref>",
    "cryptographic-timestamp": "2026-03-07T16:22:01Z"
  },
  "ledger-note": "HONEYPOT ACTIVATED — dual stream initiated — this entry exists only in hidden stream"
}
```

**The talking point:**

*"Every other system in this space solves the complicit insider problem the same way — better training, tighter supervision, whistleblower hotlines. These are people controls. They fail when the person is senior enough, trusted enough, or careful enough.*

*This system solves it architecturally.*

*Marcus Webb submitted a dismissal. He saw 'matter closed.' He believes he got away with it. What he doesn't know is that the SMR was filed the moment he submitted, the MLRO was alerted silently, and everything — his identity, his decision, his fabricated reasoning, the full workflow history — is in a ledger stream he doesn't know exists.*

*He cannot destroy the evidence. He cannot know the evidence exists. He cannot know he has been caught.*

*The honeypot doesn't prevent the act. It makes the act its own evidence.*

*For AUSTRAC — and for a bank's board — that is a categorically different capability than anything currently available."*

---



---

## Two Capabilities No Other System Has

---

### Complete Replay — Reconstruct Any Decision, Exactly

**The scenario:**
AUSTRAC opens an audit. They want to understand a no-SMR determination made 14 months ago on a transaction that has since been linked to a money laundering network. The question: *"What did your system see at the time, and why did it determine no SMR was required?"*

**What most systems can do:**
Produce a log entry showing the outcome. Maybe a timestamp. Possibly a name.

**What this system can do:**
Restore the complete execution state from the ledger at the exact moment of that decision. Re-execute every step — enrichment signals, typology scores, AI assessment, policy evaluation, officer decision, policy gate result — exactly as they ran 14 months ago. Produce a complete, step-by-step reconstruction of every decision made, every rule applied, every actor involved, with cryptographic timestamps throughout.

Not a reconstruction from memory. Not a summary. **A deterministic re-execution of the actual run.**

**How it works:**

The append-only ledger records complete input state at every decision point — not just outcomes. Every typology signal, every AI output, every policy evaluation input is preserved exactly. Replay loads that state and re-executes the workflow against it. The result is identical to the original run because the inputs are identical.

```json
{
  "replay-request": {
    "run-id": "RUN-AML-001-2024-12-14",
    "replay-type": "full-reconstruction",
    "purpose": "AUSTRAC audit response",
    "requested-by": "compliance-officer-id-0042"
  },
  "replay-result": {
    "steps-replayed": 6,
    "outcome-reproduced": "no-smr",
    "policy-rules-applied": ["auto-clear-threshold"],
    "typology-score-at-time": 4,
    "officer-decisions": 0,
    "cryptographic-match": true,
    "ledger-note": "Replay identical to original run — deterministic re-execution confirmed"
  }
}
```

**The talking point:**

*"AUSTRAC asks: 'Show us exactly what your system knew at the time and why it made that decision.' Most systems produce a log entry. This system re-runs the decision — exactly as it ran — and hands AUSTRAC a complete, step-by-step reconstruction with cryptographic proof it matches the original.*

*That is not a better audit trail. That is a categorically different level of regulatory defensibility."*

---

### Blast Radius Analysis — Know the Impact Before You Deploy

**The scenario:**
AUSTRAC releases updated typology guidance. Structuring thresholds are tightened. The compliance team needs to update the AML policy — but the question nobody can currently answer is: *"If we had applied this new policy to the last six months of transactions, what would have changed?"*

Without that answer, policy deployment is a leap of faith. Changes go live and the team discovers the impact in production — wrong decisions, missed SMRs, unexpected escalations, regulatory exposure.

**What this system does instead:**

Before a single line of policy changes in production, the team runs a blast radius analysis. The engine replays the last six months of real transaction runs against the proposed new policy. Every AI step is substituted from the ledger — the AI is not re-invoked. Every processor step is skipped — deterministic, already recorded. Only the policy evaluation steps are re-executed against the new policy.

The result is an exact diff: which decisions would have changed, which transactions would have been newly escalated, which false positives would have been eliminated, which SMRs would have been filed that weren't.

**Not a simulation. Not a sample. Every real transaction from the last six months. Exact.**

```json
{
  "blast-radius-request": {
    "proposed-policy": "aml-policy@2.0.0",
    "replay-period": "2025-09-01 to 2026-03-07",
    "transactions-replayed": 14847,
    "policy-change": "structuring-threshold reduced from 78 to 65"
  },
  "blast-radius-result": {
    "decisions-changed": 23,
    "newly-escalated-to-officer": 18,
    "newly-auto-cleared": 5,
    "additional-smrs-would-have-been-filed": 7,
    "false-positives-eliminated": 5,
    "changed-cases": [
      {
        "txn-id": "TXN-2025-11-4421",
        "original-outcome": "no-smr",
        "replayed-outcome": "threshold-met",
        "policy-rule-changed": "suspicion-threshold-met",
        "ledger-ref": "RUN-AML-4421"
      }
    ],
    "ledger-note": "Blast radius analysis complete — 14,847 transactions replayed, 23 decisions changed"
  }
}
```

**The talking point:**

*"Your compliance team wants to tighten the structuring threshold in response to new AUSTRAC guidance. Today, deploying that change means discovering its impact in production — after it's live, after decisions have been made differently, after potential regulatory exposure has already occurred.*

*This system answers the question before deployment: 'Of the 14,847 transactions we processed in the last six months, exactly 23 decisions would have gone differently. Here are the 23 cases. Here is what changed. Here are the 7 additional SMRs that would have been filed.'*

*Policy change management goes from a leap of faith to a documented, auditable, pre-validated deployment. Regulators can see that the impact was assessed before the change went live. That is a compliance capability that does not currently exist in the market."*

---

### How These Two Capabilities Connect

Replay and blast radius are not separate features. They are two expressions of the same architectural property — the append-only ledger with complete input state at every decision point.

- **Replay** looks backwards: *"What exactly happened in this run?"*
- **Blast radius** looks sideways: *"What would have happened differently under a different policy?"*

Both are deterministic. Both use real data. Both produce results that are independently verifiable by any party with access to the ledger.

Together they give a compliance team something unprecedented: **complete visibility into the past and complete confidence about the future — before anything changes.**

---

## Same Engine, Different Domain

This is the same governance engine that runs the fraud payment review demo — same route structure, same policy gate, same ledger, same unknown handler.

The only things that changed between the fraud demo and this AML demo:

| Component | Fraud Demo | AML Demo |
|-----------|------------|----------|
| Route states | Fraud-specific | AML/AUSTRAC-specific |
| Policy rules | Fraud risk thresholds | AML/CTF Act obligations |
| AI contexts | Risk assessor | Typology assessor |
| Regulatory timers | None | 3-business-day SMR gate |
| Tipping-off gate | Not applicable | Architecturally enforced |
| Domain signals | Payment risk signals | AML typology signals |

**The engine is jurisdiction-agnostic.** The same architecture supports AU (AUSTRAC SMR), US (FinCEN SAR), EU (AMLD6 STR) by swapping the policy and route definitions. The core guarantees — AI non-authoritative, unknown first-class, policy gatekeeper, append-only ledger — hold in every jurisdiction.

---

## End-to-End Provable Chain of Custody

A common challenge to ledger-based audit systems: *"A ledger entry is just a claim. How do I know the input data that fed the decision was accurate and untampered?"*

It's the right question. A ledger entry on its own is only part of the evidence picture.

The answer is architectural — and it changes the nature of the evidence claim entirely.

**The system is decomposable to any level.**

The same governance engine — same append-only ledger, same policy gates, same unknown handler — runs at every layer of the data collection process underneath the SMR decision. The engine composes recursively. Every subprocess that feeds data into the workflow can itself be governed by the same engine.

So the evidence chain is not:

```
ledger entry (SMR decision) → trust me the input data was correct
```

It is:

```
ledger entry (SMR decision)
    ↑ governed by same engine
ledger entry (typology extraction)
    ↑ governed by same engine
ledger entry (enrichment process)
    ↑ governed by same engine
ledger entry (data ingestion)
    ↑ governed by same engine
ledger entry (source system capture)
```

Every layer is governed. Every layer is auditable. Every layer is replayable. The same architecture, composed to whatever depth the institution requires.

**What this means for evidence:**

You cannot attack the SMR decision by attacking the quality or integrity of the input data — because the input data collection process has its own unbroken ledger. The chain of custody runs from raw source data capture all the way to AUSTRAC submission without a gap.

This is not a claim most compliance systems can make. Most systems have a governed decision layer sitting on top of ungoverned data pipelines. The decision is auditable. The data that fed it is not.

**The complete evidence claim:**

This system provides end-to-end provable chain of custody — from raw data capture through every enrichment and transformation step to the final regulatory submission. Every layer governed. Every layer replayable. No ungoverned data pipeline underneath a governed decision.

That is not a better audit trail. That is a categorically different standard of regulatory evidence.

**The silent assumption every compliance system makes — and this one doesn't:**

Every compliance system in the world has the same assumption buried in it: *"We trust the inputs are what they say they are."* That assumption is where fraud lives. The CBA $1B loan fraud is that assumption being exploited at scale — fraudulent documents fed into a system that trusted its inputs.

This system replaces that assumption with a guarantee:

```
Every input at every layer is either:
  → 100% classified and verified  → governed path
  → anything less                 → unknown → manual examination
```

Not "probably right." Not "passes basic validation." Not "looks reasonable." Either every signal is completely known and matched — or it is explicitly unknown and gets examined manually. There is no middle ground. There is no threshold below which something quietly passes through.

The gap that fraud exploits — the space between "we checked some things" and "we checked everything" — collapses to zero by construction. Combined with end-to-end chain of custody from raw data capture to regulatory submission, this system makes a claim no existing compliance system can match:

**There are no silent assumptions. There are no trusted inputs. There is no gap.**

---

## What AUSTRAC Can Audit

For every transaction in this system, AUSTRAC can verify:

| Regulatory Requirement | How the System Provides It |
|------------------------|---------------------------|
| Was the transaction assessed? | Ledger entry 1 — ingestion timestamp |
| Who made the suspicion determination? | Named officer in ledger entry, timestamped |
| Was reasoning documented? | Policy gate enforces reasoning before accepting decision |
| Was the SMR filed within 3 business days? | Timer gate — architecturally enforced, timer-cleared recorded |
| Was tipping off via the system prevented? | tipping-off-gate-active blocks all in-system subject communication — out-of-band attempts are detectable via audit trail and honeypot path |
| Were records retained 7 years? | Retention period commenced in terminal ledger entry |
| Could an officer suppress a suspicious matter? | No — policy rejection is itself a ledger entry that cannot be deleted |
| What if a senior officer is complicit? | Honeypot state — SMR filed silently, MLRO alerted, evidence captured before officer knows they are caught |
| Can we reconstruct a decision made 14 months ago? | Full replay — deterministic re-execution from ledger, cryptographically identical to original run |
| Can we assess the impact of a policy change before deploying? | Blast radius analysis — replay last 6 months of real transactions against proposed policy, exact diff before production |
| How do we know the input data itself was accurate and untampered? | Same engine governs every data collection subprocess underneath the decision — end-to-end chain of custody from raw source capture to AUSTRAC submission |

---

## The One-Sentence Summary

Every input is either matched to a verified policy or explicitly handled as unknown — by construction, there is no third option, and the entire history is cryptographically logged and replayable.

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