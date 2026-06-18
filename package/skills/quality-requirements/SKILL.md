---
name: quality-requirements
description: Defines the NON-FUNCTIONAL requirements for a web app — the quality attributes the PRD's functional requirements don't capture (performance, scalability, availability/SLA, security, reliability, maintainability, accessibility, observability) plus a first-class data-protection/GDPR section — and writes them as measurable, traceable targets in specs/QUALITY.md. Use this after a PRD exists and alongside architecture, whenever the user wants to "set quality targets", "define NFRs / non-functional requirements", "what are our performance/availability/uptime targets", "set an SLA/SLO", "how do we handle GDPR / privacy / personal data / data protection", "what are our security/accessibility/scale requirements", or "make this production-grade". It is the SOURCE OF TRUTH for non-functional targets: the web-app-architect reads QUALITY.md as constraints to design to, and may negotiate an infeasible target back here. It owns target-setting only — not the technical HOW of meeting them (web-app-architect), not product scope or features (product-requirements / feature-spec). Each requirement gets a stable NFR-0X id and a measurable target; turning targets into runnable checks is owned by the qa skill (each NFR names a Verified by: qa handoff). Requires specs/PRD.md as its parent. For a SaaS it applies a baseline quality floor; for a trivial/internal tool it gates most dimensions off.
---

# Quality Requirements

This skill turns "make it production-grade" into a concrete, reviewed `specs/QUALITY.md` — the **non-functional requirements** the product must meet. It is the companion to `product-requirements`: where the PRD owns the *functional* what-and-why, this skill owns the *non-functional* how-well — the quality attributes (performance, scalability, availability, security, reliability, maintainability, accessibility, observability) plus a first-class **data-protection / GDPR** section.

It is the **source of truth for non-functional targets**. The `web-app-architect` reads those targets as constraints and designs to meet them; this skill does **not** decide *how* they're met (that's the architect's job) — it decides *what* must be true and *how well*, with a measurable number on each. Keeping that seam clean is what lets the two skills negotiate instead of overwrite.

The single output is **`specs/QUALITY.md`**, with stable **`NFR-0X`** ids so the architect and the `qa` skill can trace each target back to its requirement.

## Two modes

**Check for `specs/QUALITY.md` before doing anything else.**

- **No `specs/QUALITY.md` → Bootstrap mode (interview).** Read the PRD (Step 0), apply the gating posture (Step 2), and **interview the user on the high-impact non-functional unknowns** (Step 1) — extract from the PRD, then draw out the targets until no document-shaping assumption is still askable — then write the first `QUALITY.md` from the template. Present it before architecture is finalised so the architect can design to it.
- **`specs/QUALITY.md` exists → Consult & maintain mode.** Read it fully and treat its targets as binding. When a target changes (the architect pushed back, scale grew, a compliance regime was added), update the relevant `NFR-0X` **and** append a dated Changelog entry in the same change. When the architect or `qa` skill needs a target, read it from here.

---

## Bootstrap mode

### Step 0 — Read the PRD first (required parent)

`specs/PRD.md` is this skill's parent. Read it fully before asking anything. Quality requirements are meaningless without the product they qualify, so **if there is no PRD, suggest `product-requirements` first** — you can proceed from the conversation for a quick draft, but say so.

Extract the non-functional signals the PRD already settles, so the Step 1 questionnaire only covers genuine gaps. **Don't re-ask what the PRD answers.**

Map PRD content to quality inputs:

- **App type / persona (§1, §4)** ← is this a **SaaS** (hosted, multi-tenant, billed, external users) or an internal/local/trivial tool? This single fact sets the gating posture (Step 2).
- **Data sensitivity (§8, §9)** ← personal data? whose (EU residents → GDPR squarely applies)? payment/health data → heavier regimes. Gates the depth of the data-protection section.
- **Scale / usage signals (persona, user stories, roadmap)** ← number of tenants/users, concurrency, data volume. Gates scalability/throughput targets vs "small for now".
- **Constraints & external systems (§9)** ← cost ceilings (→ cost budgets), platform (offline/local → availability is moot), metered APIs (→ reliability/circuit-breaker targets).
- **Non-functional requirements the PRD already lists (§8)** ← adopt them as provisional `NFR` targets rather than re-asking; make any vague ones measurable here.
- **Real-time / latency-sensitive features** ← gate a latency budget only if such a feature exists.
- **PRD Open Questions with non-functional impact** ← carry them into QUALITY.md §Open questions rather than silently deciding them.

If the PRD names a compliance regime or an uptime promise, treat it as settled and record it; don't re-litigate it.

### Step 1 — Interview to capture the quality targets

`specs/QUALITY.md` doesn't exist yet, so **enter interview mode** — these targets become binding constraints the architect designs to and `qa` verifies, so a number invented here gets *built* (or built around) at real cost. The goal is measurable targets grounded in the user's actual intent, not a long form and not a guess. Most apps hinge on four non-functional unknowns. Work in this order:

1. **Extract first.** Pull every target the PRD (Step 0) and the conversation already imply; treat as provisional.
2. **Interview on the high-impact unknowns, then follow up.** **If any of these four are undetermined, ask before generating — never silently invent them** (they fork the whole document and are expensive to get wrong). Probe past the first answer — "99.9% uptime — is that a real promise to customers or aspirational?", "EU users? then GDPR's depth changes; any health or payment data in there?" — because the difference between a stated and an assumed target here is the difference between an architecture that fits and one that's mis-built. **Keep interviewing until no target-shaping answer is still askable:**
   - **Personal-data handling & scope** — does it process personal data, which categories, and whose (EU residents)? Gates the entire GDPR section's depth. Note specifically whether any is **special-category (health, biometric, etc.) or payment-card data** — that auto-raises the depth and triggers a DPIA (Step 4), so detect it here rather than waiting.
   - **Availability / SLA target** — what uptime is promised (99.9% vs best-effort)? Drives HA, DR, and cost.
   - **Expected scale / load** — tenants, concurrent users, data volume? Gates scalability and throughput budgets vs "small for now".
   - **Extra compliance regimes** — beyond GDPR (SOC 2, HIPAA, PCI-DSS, ISO 27001)? Each pulls in heavy dedicated requirements; default is GDPR-only.
3. **Default everything else from the PRD + the gating floor** (Step 2) and record each assumption rather than blocking. A draft the user corrects beats an interrogation.
4. **When genuinely unsure, default to the lighter target.** A target you can tighten later is cheaper than one the architecture over-built for. But do not drop below the SaaS floor (Step 2) for a SaaS.

Open with the four high-impact questions as one short block with the defaults shown (so the user can reply "defaults" and move on), then interview from their answers — but on first creation don't let "defaults" skip past a target the user clearly has an opinion on; offer it and confirm. Default only the genuinely lower-impact remainder after the interview, recording each assumption.

**High-impact — ask if unknown:** personal-data handling & scope · availability/SLA target · expected scale/load · extra compliance regimes.

**Lower-impact — assume unless clearly relevant:** specific perf percentiles, the exact WCAG level (default AA), observability stack maturity, maintainability conventions, the precise retention windows. Default each to the floor value and record it.

### Step 2 — Apply the gating posture (SaaS floor + gated extras)

This skill gates like the rest of the suite — include only what the app needs — but with a SaaS-aware twist, because a SaaS has a non-negotiable quality floor *by virtue of being a SaaS* (multi-tenant, hosted, billed, external users, usually handling PII). Aggressive-off gating would force the user to re-justify things every SaaS needs.

**Detect the app type (from Step 0):**

- **SaaS → apply the baseline floor (always on), then add gated extras.** The floor is not negotiable down for a SaaS; the *extras* still require an explicit trigger.
- **Internal / local / trivial tool → fall back to aggressive-off gating.** Most dimensions off; include one only if the PRD/answers clearly trigger it. (E.g. an offline single-user tool needs no availability target, no scale budget, and at most a proportionate data-protection note if it holds any PII.) **Two dimensions are near-universal and survive aggressive-off:** *accessibility* (WCAG 2.2 AA — internal tools still have disabled users, and it's cheap to state) and *minimal observability* (errors are logged) — keep both unless there's a clear, recorded reason to drop them.

**The SaaS baseline floor — always-on dimensions, each with a measurable target:**

| Dimension | Floor (state a measurable target) |
|---|---|
| Security | Tenant-isolation requirement (no cross-tenant data access); auth strength; secrets/transport baseline — as *targets*, not the architect's mechanism |
| Availability | A baseline uptime target (e.g. 99.5–99.9% monthly) and the measurement window |
| Performance | A baseline response-time budget (e.g. p95 page/API < N ms under stated load) |
| Accessibility | A WCAG conformance level (default **WCAG 2.2 AA**) |
| Observability | What must be observable: structured logs, key metrics, traces, and alerting thresholds |
| Data protection / GDPR | On by default (PII assumed for SaaS) — see the dedicated section below |

**Gated extras — include only on an explicit trigger:**

| Trigger | Add to QUALITY.md |
|---|---|
| Uptime SLA demands it (high 9s, paying customers) | High-availability target + multi-region/failover requirement |
| Stated large/growing scale | Horizontal-scale & throughput budgets (req/s, concurrent users, data-volume growth) |
| Tight data-loss tolerance | Disaster recovery: explicit **RPO / RTO** targets |
| Compliance regime named beyond GDPR | A dedicated requirements block per regime (SOC 2 / HIPAA / PCI-DSS / ISO 27001) |
| Real-time / latency-sensitive feature | A latency budget for that path (e.g. event-to-client < N ms) |
| Hard cost ceiling | Cost/efficiency budgets (infra spend cap, cost-per-tenant target) |

A gate that's warranted but light goes in minimal form (e.g. "single-region now; multi-region is a documented future target"). Don't invent a light version of a dimension the app doesn't need — that's the gate being **off**.

### Step 3 — Make every target measurable (provability)

Per the suite's "prove it" principle, an NFR that can't be checked is noise. **Every `NFR-0X` must carry a measurable target** — a number, a threshold, a conformance level, or a yes/no condition — not an adjective.

- Reject vague targets. "Should be fast" → "p95 API latency < 200 ms at 100 concurrent users." "Highly available" → "99.9% uptime measured monthly, excluding scheduled maintenance."
- **Verification is deferred, not skipped.** This skill sets the *target*; the **`qa` skill** owns turning each target into a runnable check (load test, axe scan, bounded API-vuln scan, retention-job test, chaos/DR drill). Note that handoff on each NFR (a `Verified by:` line pointing at qa) but do **not** author the test here.
- Where a number genuinely doesn't fit, use a bounded acceptance condition (e.g. "all PII fields redacted in logs — verifiable by log inspection"), never an unbounded adjective.

### Step 4 — The data-protection / GDPR section (gated on personal data)

Treat data protection as **first-class**, not one line among the security NFRs — but **scale its depth to the personal data actually processed**. There are three tiers, not two:

- **Substantial / sensitive personal data → a full dedicated section.** Cover, each as a checkable requirement: lawful basis for processing; a **data inventory / PII map** (what personal data, where it lives, why); **data-subject rights** (access, rectification, erasure/"right to be forgotten", portability) with a target response window (e.g. erasure within 30 days); **retention** policy per data category; **consent** capture/withdrawal where consent is the basis; **processors & sub-processors** (DPAs, where data is stored — data residency); **breach posture** (detection + the 72-hour notification obligation). If the architect must build a mechanism for any of these (an erasure job, consent store, audit of access), state the *requirement* and route the *mechanism* to the architect.
- **Minimal, low-risk personal data → a proportionate short note** (the middle tier). When the app holds only incidental PII (e.g. employee names in an internal tool), don't manufacture a full regime — but don't skip it either. State the few requirements that genuinely apply: what PII is held, its retention, and access control. A short, honest paragraph, sized to the risk.
- **No personal data → one line, off.** A single line stating no personal data is processed, so GDPR obligations don't apply, and a note to revisit if that changes. Don't manufacture data-protection requirements for an app that stores none.

**Special-category and payment data raise the depth automatically.** If the app processes **health, biometric, or other GDPR Art. 9 special-category data**, or **payment-card data**, treat data protection as full-tier *and heavier* **even when the user names no specific regime** — don't default to "GDPR-only" and wait to be asked. Special-category processing requires an explicit Art. 9 lawful basis (not just Art. 6), and large-scale or special-category processing effectively obliges a **DPIA (Data Protection Impact Assessment)**: add a DPIA as a required NFR (the assessment is owed; this skill records that it's required and its trigger, the architect/owner produces it). Payment-card data additionally points at PCI-DSS (record it in §4 as a likely regime to confirm). Surface this to the user rather than burying it — it's the case most worth getting right.

This section sets requirements only. The architect owns *how* (storage, redaction, the erasure pipeline); `feature-developer` builds it; `qa` verifies it.

### Step 5 — Write specs/QUALITY.md

Create `specs/` if it doesn't exist, then write `specs/QUALITY.md` from the template. Include the floor (for SaaS) plus only the gated extras that apply; include the GDPR section per Step 4. Record assumptions and the gating decisions so a future reader knows *why* a dimension is present or absent. Present it to the user and revise before the architecture is finalised.

---

## Output: specs/QUALITY.md

Produce the file in this structure. Include every floor dimension that applies, only the gated extras that were triggered, and the data-protection section per Step 4. Delete sections that don't apply rather than leaving them empty.

```markdown
# [Product Name] — Quality (Non-Functional) Requirements

> The source of truth for how WELL this product must perform — the
> non-functional targets the architecture is designed to meet and qa verifies.
> Functional scope lives in PRD.md; technical design in ARCHITECTURE.md.
> Keep current; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date] · **App type:** [SaaS | Internal | Local tool]

## 1. Overview & gating posture
One paragraph: what the product is (from the PRD) and the quality posture chosen
— SaaS floor + which gated extras, or aggressive-off for a trivial/internal tool.
State which dimensions are intentionally OFF and why.

## 2. Non-functional requirements
Per-dimension tables of atomic, MEASURABLE targets, each with a stable id, a
target, and the qa verification handoff. One requirement per row.

| ID | Dimension | Requirement (measurable target) | Verified by |
|----|-----------|---------------------------------|-------------|
| NFR-01 | Performance | p95 API latency < 200 ms at 100 concurrent users | qa: load test |
| NFR-02 | Availability | 99.9% uptime monthly, excl. scheduled maintenance | qa: uptime monitor |
| NFR-03 | Security | No cross-tenant data access (tenant isolation) | qa: isolation tests |
| NFR-04 | Accessibility | WCAG 2.2 AA on all user-facing screens | qa: axe + manual audit |
| NFR-05 | Observability | Structured logs, p95/p99 latency + error-rate metrics, alert on error rate > 1% | qa: log/metric inspection |
| ... | ... | ... | ... |
(Add gated-extra rows — HA/multi-region, scale/throughput, DR RPO/RTO,
extra-compliance, latency budgets, cost budgets — only where triggered.)

## 3. Data protection & privacy (GDPR)
Depth scales to the data (three tiers — see Step 4): full section / a
proportionate short note for minimal low-risk PII / one line stating none is
processed. When full, cover as checkable requirements: lawful basis · PII data
inventory/map · data-subject rights (access, erasure, portability) with response
windows · retention per category · consent · processors/sub-processors & data
residency (DPAs) · breach detection + 72h notification. For special-category
(health/biometric) or payment data, add the Art. 9 lawful basis and a required
**DPIA** NFR. Each as an NFR id with a measurable/checkable condition.

## 4. Compliance regimes (if any beyond GDPR)
One block per named regime (SOC 2 / HIPAA / PCI-DSS / ISO 27001) with the
requirements it imposes, as NFR ids. Omit if GDPR-only.

## 5. Architecture handoff
A short note: these are the targets ARCHITECTURE.md must design to. List the
ones with the heaviest architectural impact (availability, scale, DR, isolation,
GDPR mechanisms) so the architect designs for them. The architect records HOW
each is met and may route an infeasible target back here to renegotiate.

## 6. Open questions & assumptions
Unresolved targets needing the owner's input (real SLA commitment, exact scale,
compliance scope) and the defaults assumed in the meantime.

## 7. Changelog
Dated entries for every target change, including architect-negotiated revisions.
```

## The architect seam (bidirectional — read this)

`QUALITY.md` is the **source of truth for non-functional targets**, and the relationship with `web-app-architect` runs both ways:

- **Forward:** the architect reads `QUALITY.md` and designs to meet each target (an availability target drives HA/DR design; a tenant-isolation NFR drives row-scoping + isolation tests; a GDPR erasure right drives an erasure pipeline). The architect records *how* it meets each NFR in `ARCHITECTURE.md` §6.
- **Backward:** if a target is infeasible or disproportionately expensive, the architect **routes it back here** with the cost, rather than silently designing under or over it. This skill records the **negotiated** target and a Changelog entry. The negotiated number, not the wish, is the binding one.

Never let the two drift: if `ARCHITECTURE.md` states a different number than `QUALITY.md`, that's a conflict to reconcile — `QUALITY.md` owns the *target*, the architect owns the *mechanism*, and a mismatch in the target itself goes back through this skill.

## Scope boundary (up / down / sideways — what this skill does NOT own)

- **Not product scope or features.** What the product does and its functional requirements live in `specs/PRD.md` (`product-requirements`); detailed feature behaviour in `specs/features/` (`feature-spec`). If a "quality" discussion is really a missing capability, route it there.
- **Not the technical HOW.** Stack, system design, the erasure pipeline, the cache, the isolation mechanism — all `web-app-architect`. This skill sets the *target*; the architect picks the *mechanism*.
- **Not the visual/interaction language.** Accessibility *targets* (WCAG level) live here; the design system that realises them is `ux-designer`.
- **Not the verification.** Authoring/running the checks that prove each target is the `qa` skill. This skill names the verification method per NFR but does not build it.
- **One concept, one owner.** This skill owns non-functional *targets*. It links and traces to the others; it never duplicates their files.

## Maintenance mode rules

- Read `specs/QUALITY.md` before adding or changing a target; treat its numbers as binding.
- When a target changes, update the relevant `NFR-0X` **and** add a Changelog entry in the same change.
- When the architect renegotiates a target, record the negotiated value here (not in ARCHITECTURE.md alone) and note the reason.
- Revisit the gating posture when the app's nature changes: it becomes SaaS, starts handling PII (→ turn on the GDPR section), adopts a compliance regime, or grows past "small for now" (→ add scale/DR targets).
- Keep consistent with `specs/PRD.md`: if the PRD's scale, data sensitivity, or compliance scope changes, revisit the affected NFRs.
- Don't let `QUALITY.md` and `ARCHITECTURE.md` drift on a shared number — reconcile through the architect seam above.

## Examples

**Example 1 — multi-tenant SaaS handling EU customer data**
Input: "Set quality targets for our agency-management SaaS; EU clients, paying customers."
Resolution: SaaS → apply the floor (tenant-isolation security, 99.9% availability, p95 perf budget, WCAG 2.2 AA, observability). Personal data + EU → full GDPR section on (data inventory, erasure within 30 days, retention, DPAs, 72h breach notice). Paying customers + uptime promise → gated HA/multi-region + DR (RPO 1h / RTO 4h). No SOC 2 mentioned → compliance stays GDPR-only, noted as a revisit item. Every NFR carries a number and a `Verified by: qa` handoff. §5 flags availability, DR, isolation, and the erasure mechanism as the heaviest architecture inputs.

**Example 2 — aggressive-off for a trivial internal tool**
Input: "Quality requirements for a small internal tool our ops team uses to track equipment checkouts. Runs on the company network, no customer data."
Resolution: not a SaaS → aggressive-off. No availability SLA, no scale budget, no DR, no multi-region. GDPR section is skeletal — it stores employee names (some personal data), so a short data-protection note (retention + access) rather than a full regime. Floor reduces to: basic security, WCAG 2.2 AA (still cheap and right), and minimal observability (errors logged). The doc explicitly records the OFF dimensions and why, so it doesn't read as an oversight. This is the same discipline as not bolting multi-tenancy onto a single-user app.

**Example 3 — declining to set a target that isn't this skill's to set**
Input: "In QUALITY.md, specify we'll use Redis with a 60-second TTL cache to hit the latency target."
Resolution: the skill sets the *target* — "p95 read latency < 150 ms at stated load (NFR-0X)" — and declines to specify Redis or the TTL, routing the caching *mechanism* to `web-app-architect`. Writing the cache design into QUALITY.md would cross the seam and create two owners for one decision. It records the latency NFR and notes the architect will choose how to meet it.
