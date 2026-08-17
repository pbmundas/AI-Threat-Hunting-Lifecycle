# AI Threat Hunting Lifecycle & Methodology
---

## 1. Purpose & Scope

### 1.1 Purpose
This framework defines a repeatable, operational lifecycle for proactively hunting adversary activity targeting an organization's AI systems large language model (LLM) applications and internal copilots, autonomous agents and MCP servers, traditional machine learning models (fraud, risk, recommendation), and third-party AI features embedded within SaaS tools.

It exists to close a specific, acknowledged gap: MITRE ATLAS gives us a taxonomy of what adversaries do to AI systems, but not a methodology for how a hunter goes from that taxonomy to a validated detection in a real environment. This document is that missing methodology.

### 1.2 Scope
This is **Face A** of a two-part program hunting attacks directed *at* AI systems (data poisoning, prompt injection, model extraction, agent/MCP abuse, etc.). It does not cover **Face B** (AI used *as* a weapon against conventional infrastructure AI-generated phishing, AI-automated recon, deepfake fraud), which is addressed in a separate companion framework once this one is operational.

### 1.3 In-Scope AI System Categories
| Category | Examples |
|---|---|
| LLM-based apps / internal copilots | Internal chatbots, coding copilots, customer-support LLM apps, RAG-based knowledge assistants |
| Autonomous agents / MCP servers | Agents with tool-invocation capability, multi-step task automation, MCP server integrations |
| Traditional ML models | Fraud detection, credit risk scoring, recommendation engines, anomaly detection models |
| Third-party AI features embedded in SaaS | AI features inside CRM, HR, productivity, or security tools the org doesn't directly control |

### 1.4 Foundational Reference
This framework builds on **MITRE ATLAS** (Adversarial Threat Landscape for Artificial-Intelligence Systems) as its taxonomic base. ATLAS techniques and case studies are referenced by ID throughout hunting hypotheses; this document does not duplicate the ATLAS matrix, it operationalizes it.

---

## 2. Governing Principles

1. **A technique ID is not a detection.** As with ATT&CK, an ATLAS technique may have many procedures; one detection rule rarely covers a technique fully. Coverage claims must specify which procedure(s) are actually covered.
2. **No hunt without a data-source check.** A hypothesis is not ready for execution until visibility has been confirmed. This is a hard gate, not a suggestion (see Stage 4).
3. **AI systems are probabilistic false-positive reasoning must account for model variance.** A hunting hypothesis must explicitly state what distinguishes malicious signal from ordinary model/agent variance, not just what the malicious signal looks like.
4. **Validate before you trust a detection.** No detection is considered production-ready until it has been tested against a safe, controlled simulation of the actual technique (Stage 6). Undetected-until-tested rules are treated as unvalidated, not as coverage.
5. **The taxonomy is a living document.** Given the pace of change in this space, this framework assumes quarterly minimum taxonomy review, not annual.

---

## 3. Roles & Ownership (RACI Summary)

| Role | Responsibility |
|---|---|
| **AI Threat Hunting Lead** | Owns the framework, prioritization, and quarterly taxonomy review |
| **Threat Hunter(s)** | Execute Stages 3–7 for assigned hypotheses |
| **CTI Analyst** | Owns Stage 2 (intel intake and ATLAS mapping) |
| **Detection Engineer** | Owns Stage 7 (rule codification) and supports Stage 6 (validation) |
| **AI/ML Platform Owner** | Owns Stage 1 asset inventory accuracy and Stage 4 telemetry enablement for their systems |
| **SOC Manager** | Consumes Stage 8 outputs; owns operational handoff of validated detections |

---

## 4. The Eight-Stage Lifecycle

### Stage 1 Scoping & Asset Inventory
**Objective:** Establish and maintain an accurate, living inventory of every AI system in scope, since AI is the fastest-growing category of shadow IT it frequently enters an environment embedded inside SaaS products rather than through a formal procurement or engineering process.

**Key activities:**
- Discover AI systems via network/API traffic analysis, SaaS app inventory review, procurement records, and direct engineering team interviews
- Classify each asset by category (per Section 1.3), data sensitivity, business criticality, and exposure (internal-only vs. customer-facing vs. third-party-embedded)
- Record ownership, and confirm an accountable owner exists for every asset

**Output/artifact:** AI Asset Inventory Register (see companion workbook)
**Cadence:** Continuous discovery; formal full review monthly
**Gate to next stage:** An asset must appear in the register before any hypothesis targeting it can be approved for hunting

---

### Stage 2 Intelligence Intake & ATLAS Mapping
**Objective:** Translate external CTI (vendor reports, ATLAS case studies, academic red-team research, incident post-mortems) into ATLAS technique/sub-technique mappings relevant to the org's actual asset inventory.

**Key activities:**
- Monitor ATLAS changelog and case-study updates (recommend at minimum bi-weekly given release cadence)
- Map incoming intel to specific ATLAS tactic/technique IDs
- Cross-reference the technique against Stage 1's asset inventory is this technique even relevant to something we run?
- Flag techniques with no current ATLAS coverage for potential Face-B or internal taxonomy extension

**Output/artifact:** Prioritized technique backlog, each entry tagged with relevant asset category from Stage 1
**Cadence:** Continuous intake; prioritization review bi-weekly
**Owner:** CTI Analyst, in collaboration with AI Threat Hunting Lead

---

### Stage 3 Hypothesis Generation
**Objective:** Convert a prioritized technique into a specific, testable, falsifiable hunting hypothesis.

**Required hypothesis template** (all four fields mandatory):
1. **If-then claim:** "If [technique] is occurring against [asset], we would expect to observe [specific signal] in [specific telemetry]."
2. **Distinguishing condition:** "This differs from normal model/agent variance because [specific reasoning] e.g., frequency, sequence, or magnitude that ordinary usage does not produce."
3. **Scope:** Which specific asset(s) from the Stage 1 register this hypothesis applies to.
4. **ATLAS reference:** Technique/sub-technique ID and, where available, case study reference.

**Output/artifact:** Documented hypothesis, added to the hunt backlog
**Cadence:** Per prioritized technique from Stage 2
**Owner:** Threat Hunter

---

### Stage 4 Data Source & Telemetry Readiness Check
**Objective:** Confirm the organization actually has the visibility required to test the hypothesis before committing hunt time. This is a hard gate hypotheses that fail this check are not hunted, they are routed to a telemetry-gap backlog instead.

**Required telemetry categories to check per hypothesis:**
- Prompt / query / completion logs
- Agent tool-invocation and MCP server call logs
- Embedding / vector database access logs
- Model inference request/response metadata (rate, source, parameters)
- Training pipeline and data-ingestion audit logs (for poisoning-related hypotheses)
- Standard infrastructure telemetry (auth, network, API gateway) where the AI system sits behind conventional infrastructure

**Output/artifact:** Go/no-go decision per hypothesis; telemetry-gap entries routed to AI/ML Platform Owner for remediation
**Cadence:** Per hypothesis, before Stage 5 begins
**Owner:** Threat Hunter, with AI/ML Platform Owner sign-off on telemetry availability

---

### Stage 5 Hunt Execution
**Objective:** Execute the hunt against confirmed telemetry sources.

**Key activities:**
- Query relevant logs/telemetry for the hypothesized signal
- Document all findings, including negative results (a hunt that finds nothing is still a valid, recordable outcome see Governing Principle in Section 2)
- Escalate any confirmed malicious activity to IR immediately, independent of the rest of this lifecycle

**Output/artifact:** Hunt findings record (positive, negative, or inconclusive)
**Cadence:** Per approved hypothesis
**Owner:** Threat Hunter

---

### Stage 6 Validation via Controlled Simulation
**Objective:** Confirm that a candidate detection would actually fire against the real technique, not just against the hunter's mental model of it.

**Key activities:**
- Where available, use existing adversarial ML testing tools to safely simulate the technique in a controlled, non-production environment
- Where no mature tooling exists for a given technique currently the case for much of the agentic AI attack surface document a manual, reviewed simulation procedure instead, and flag it explicitly as manually validated rather than tool-validated
- Confirm the candidate detection fires on the simulated technique and does not fire on a documented set of benign/normal-variance scenarios

**Output/artifact:** Validation record (tool-validated or manually-validated), attached to the candidate detection
**Cadence:** Before any detection moves to Stage 7 production status
**Owner:** Threat Hunter, with Detection Engineer support

**Honest note for this framework version:** a mature, technique-by-technique adversarial simulation library (an "Atomic Red Team for AI") does not yet broadly exist. Until it does, this stage will lean heavily on manual, carefully reviewed simulation track this as a known maturity gap, not a process failure.

---

### Stage 7 Detection Codification
**Objective:** Turn a validated hunt into a standing, production detection.

**Key activities:**
- Write the detection logic in the org's standard detection-as-code format
- Explicitly document which procedure(s) of the ATLAS technique this detection covers and which it does not
- Tag the detection with its ATLAS technique/sub-technique ID and Stage 1 asset scope for future coverage-mapping
- Hand off to SOC with a triage playbook (what the alert means, what to check, escalation path)

**Output/artifact:** Production detection rule + triage playbook
**Cadence:** Per validated hunt
**Owner:** Detection Engineer

---

### Stage 8 Reporting & Lessons Learned
**Objective:** Close the loop feed findings back into taxonomy refinement, coverage tracking, and program prioritization.

**Key activities:**
- Update the technique coverage map (which ATLAS techniques/procedures are covered, partially covered, or not coverable given current visibility)
- Document lessons learned, especially telemetry gaps surfaced during Stage 4 that need longer-term remediation
- Feed newly observed adversary behavior not yet in ATLAS back into Stage 2 as candidate additions to the internal taxonomy extension
- Report rollups to leadership: hypotheses tested, coverage gained, confirmed findings, open telemetry gaps

**Output/artifact:** Updated coverage map, lessons-learned log, leadership report
**Cadence:** Per hunt cycle, with a formal quarterly rollup
**Owner:** AI Threat Hunting Lead

**Feedback loop:** Outputs from Stage 8 feed directly back into Stage 2's prioritization backlog, closing the lifecycle.

---

## 5. Maturity Model

Use this to self-assess program maturity and set realistic near-term goals.

| Level | Description |
|---|---|
| **Level 0 Unaware** | No formal AI asset inventory; AI risk not distinguished from general IT risk |
| **Level 1 Aware** | Asset inventory exists but is manually maintained and incomplete; ATLAS mapping is ad hoc |
| **Level 2 Operational** | Full 8-stage lifecycle in use; hypotheses documented; Stage 4 gate enforced |
| **Level 3 Validated** | Stage 6 validation consistently performed (tool- or manually-validated) before any detection reaches production |
| **Level 4 Adaptive** | Quarterly taxonomy review formalized; internal extensions to ATLAS tracked and contributed back to the community where appropriate; Face B program integrated |

Most organizations starting this framework should target **Level 2 within the first quarter**, and treat Level 3 as the honest, achievable ceiling until adversarial-simulation tooling for AI matures further industry-wide.

---

## 6. Governance & Versioning

- This document is versioned (currently v1.0). Material changes to the lifecycle stages require AI Threat Hunting Lead sign-off.
- The technique/hypothesis workbook (companion artifact) is reviewed and updated on the same quarterly cadence as Stage 8's formal rollup.
- Any internal taxonomy extensions (techniques observed but not yet in ATLAS) should be tracked separately and, where appropriate, submitted back to the ATLAS project consistent with its open, community-contribution model.

---

## 7. Companion Artifacts

This document is designed to be used alongside two companion artifacts:
1. **AI Asset Inventory Register** (Stage 1 template)
2. **ATLAS-Scoped Hunting Hypothesis Workbook** (Stages 2–4 working document, pre-populated with priority techniques across all four in-scope asset categories)

