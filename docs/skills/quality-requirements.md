# `quality-requirements`

> Defines the **non-functional** requirements — *how well* the product must perform — as measurable, traceable targets.

[← Back to the overview](../../README.md) · **Stage:** between the PRD and the architect (optional) · companion to [`product-requirements`](product-requirements.md)

---

## What it does

Turns "make it production-grade" into a concrete, reviewed `specs/QUALITY.md`. It is the companion to [`product-requirements`](product-requirements.md): where the PRD owns the *functional* what-and-why, this skill owns the *non-functional* how-well — the quality attributes (performance, scalability, availability/SLA, security, reliability, maintainability, accessibility, observability) plus a **first-class data-protection / GDPR section**.

It is the **source of truth for non-functional targets**. It sets *what must be true and how well* with a measurable number on each; it does **not** decide *how* they're met (that's the architect). Every requirement gets a stable `NFR-0X` id and a measurable target — no adjectives.

It works in **two modes**: bootstrap (write the first `QUALITY.md` from the PRD + gating posture) and consult & maintain (update an `NFR-0X` + changelog when a target changes).

**SaaS-aware gating.** A SaaS gets an always-on **baseline floor** (tenant isolation, baseline availability, perf budget, WCAG 2.2 AA, observability, GDPR-on) with heavier dimensions (HA/multi-region, scale/throughput, DR RPO/RTO, extra compliance regimes, real-time latency, cost budgets) gated on explicit triggers. A trivial/internal tool falls back to aggressive-off — though *accessibility* and *minimal observability* survive even that.

**GDPR has three tiers**, scaled to the data actually processed: a full dedicated section for substantial/sensitive PII, a proportionate short note for incidental PII, one line stating "none" otherwise. Special-category (health/biometric) or payment data automatically raises the depth and triggers a required DPIA.

## Sample prompts

- "Set quality targets for our agency-management SaaS; EU clients, paying customers."
- "Define the NFRs / non-functional requirements for this app."
- "What should our performance / availability / uptime targets be? Set an SLA."
- "How do we handle GDPR / privacy / personal data / data protection?"
- "Make this production-grade." / "What are our security / accessibility / scale requirements?"

> On first creation it *interviews* you, leading with four high-impact questions — personal-data handling & scope · availability/SLA · expected scale/load · extra compliance regimes — and probing the answers ("is that uptime a real promise or aspirational?", "EU users, and any health/payment data?") until no target-shaping answer is still worth asking about, then defaults the rest from the PRD and the floor.

## What it produces

| Output | Description |
|---|---|
| `specs/QUALITY.md` | The source of truth for non-functional targets, as stable `NFR-0X` ids each with a measurable target and a `Verified by:` handoff. |

Structured into: Overview & gating posture (states which dimensions are intentionally OFF and why) · **Non-functional requirements (per-dimension `NFR-0X` tables)** · **Data protection & privacy (GDPR)** · Compliance regimes (if any beyond GDPR) · Architecture handoff · Open questions & assumptions · Changelog.

## Dependencies & consumers

**Reads:**

| Input | For… |
|---|---|
| `specs/PRD.md` | **required parent** — app type (SaaS vs internal), data sensitivity, scale, constraints, compliance signals |

**Consumed by:**

| Consumer | Uses the targets for… |
|---|---|
| [`web-app-architect`](web-app-architect.md) | reads each target as a **constraint to design to**, records *how* in `ARCHITECTURE.md` |
| [`qa`](qa.md) | turns each `NFR-0X` `Verified by: qa` handoff into a runnable check (load test, axe scan, bounded API-vuln scan, retention-job test…) |

**The architect seam is bidirectional:** forward, the architect designs to each target and records the mechanism; backward, an infeasible/expensive target routes *back here* to renegotiate. `QUALITY.md` owns the **target**, the architect owns the **mechanism**, and the two must never drift on a shared number.

## Scope boundary

Owns non-functional **targets only.** Not product scope or features ([`product-requirements`](product-requirements.md) / [`feature-spec`](feature-spec.md)); not the technical *how* of meeting them ([`web-app-architect`](web-app-architect.md) — e.g. it sets "p95 < 150 ms", it does *not* specify Redis with a 60s TTL); not the visual language that realises accessibility ([`ux-designer`](ux-designer.md)); not the verification ([`qa`](qa.md) — it names the `Verified by` method per NFR but does not build the check).
