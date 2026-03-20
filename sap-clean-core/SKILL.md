---
name: sap-clean-core
description: >
  Use this skill for ANY question or task related to SAP Clean Core — strategy, extensibility
  decisions, remediation planning, governance, integration patterns, and upgrade readiness.
  Trigger whenever the user: asks "is this clean core compliant?"; wants to classify a Z-object
  as Level A/B/C/D (2025 model) or by C0/C1/C2 release contract; needs help choosing between
  Key User Extensibility, ABAP Cloud BAdI, RAP, or BTP side-by-side; asks about ATC findings,
  ABAP Cloud profile, or transport-blocking violations; wants a clean core remediation roadmap;
  asks about OData vs RFC, Business Events, or Event Mesh integration patterns; mentions
  "upgrade readiness", "RISE with SAP", "Cloud ALM", or "custom code analytics"; asks about
  ARB, SSB, TDS/CCS KPIs, Boy Scout Principle, or the two-layer IT model. Always use this
  skill for clean core topics — even for quick decisions like "can I use this function module?"
---

# SAP Clean Core Skill

A comprehensive reference for SAP Clean Core architecture, extensibility, governance, and remediation — aligned to the official SAP Clean Core Extensibility White Paper and the August 2025 A-D Extensibility Level model.

**Pandora context**: Saurabh works at Pandora on SAP S/4HANA **2022 FPS02** (DS4/QS4/PS4), running RISE with SAP Private Cloud Edition. He is actively building Clean Core compliance tooling (`ZTCC_CC_UPLOADER`, `ZR_CC_WRICEF_ALV_FILT3`, `ZTR_CLOUD_READINESS`) and uses the `abap-adt` MCP for direct system interaction.

---

## Quick Decision Framework — Use This First

For any extension or custom code question, apply in order:

| Step | Question | If YES → Stop |
|------|----------|---------------|
| 1 | Can standard SAP configuration handle this? | Configure it |
| 2 | Can Key User Extensibility handle this? (Custom Fields & Logic app) | Use Key User |
| 3 | Is there a released BAdI / enhancement spot? | Implement ABAP Cloud BAdI |
| 4 | Does this need a new business object or API? | Build with RAP (ABAP Cloud) |
| 5 | Complex UI, cross-system logic, or AI/ML? | Side-by-side BTP extension |
| 6 | None of the above? | Challenge requirement. File SAP influence request |

**Golden Rule**: Never modify SAP standard objects. Only consume C1 (released) APIs.

---

## The A-D Extensibility Levels (August 2025)

| Level | Label | API Type | Upgrade Risk | New Dev? | Governance |
|-------|-------|----------|-------------|----------|------------|
| **A** | Gold Standard | Released C1 APIs only (ABAP Cloud / BTP) | None — SAP guarantee | **Mandatory for all new dev** | No special approval |
| **B** | Conditionally OK | Classic APIs — documented, stable (user exits, classic BAdIs, ALV enhancements) | Low-Medium | Acceptable transitionally | ARB awareness; log in inventory |
| **C** | Conditionally Clean | SAP internal objects (C0, not released) | High | Avoid — exception only | Time-boxed remediation plan required |
| **D** | Not Clean Core | System modifications, implicit enhancements, direct write to SAP tables | Critical | **NEVER** | Emergency remediation — zero tolerance |

**ATC mapping**: Level D = Priority 1 (BLOCKS transport) | Level C = Priority 2 (Warning, can block) | Level B = Priority 3 (Informational) | Level A = No finding

---

## Release Contracts (C0/C1/C2)

| Contract | Meaning | Safe? |
|----------|---------|-------|
| **C1** — Use System-Internally | Released for customer use. Stable across upgrades. | **YES** |
| **C0** — Not Released | SAP internal. May change anytime. ABAP Cloud compiler blocks it. | **NEVER** |
| **C2** — Released for Key Users | Config-level extensions via Fiori apps. Limited scope. | YES (Key User only) |

---

## The Four Extensibility Layers

### Layer 1: Key User Extensibility (No code required)
- Custom fields on standard screens and reports
- Custom business objects (lightweight entities)
- Custom logic with SAP formula builder
- Custom analytical queries and workflows
- **Rule**: If it can be done here, it MUST be done here. Never use developer extensibility when key user tools suffice.

### Layer 2: ABAP Cloud Development (On-stack, C1 only)
- **BAdIs**: Primary extension hook. SAP pre-defines extension points; you implement the interface. Upgrade-safe.
- **RAP (RESTful Application Programming Model)**: Modern way to build custom business objects and expose as OData. Use RAP for any new custom BO.
- **CDS Views**: For data access — only released CDS views, never direct table SELECTs on C0 tables.
- **Restrictions**: No C0 API calls, no obsolete ABAP statements, must activate ABAP Cloud profile in ADT.

### Layer 3: Classic ABAP (Legacy — avoid, plan to retire)
- Z-programs, exits, legacy BAdIs
- Classify as Level B (classic exits/BAdIs) or Level C/D (if C0 or modifications)
- Target: refactor to ABAP Cloud or retire

### Layer 4: Side-by-Side BTP (For complex, cross-system, AI scenarios)
- CAP (Cloud Application Programming model), SAP Build, Fiori custom apps
- Always connects back to S/4HANA via released OData APIs or Business Events — never direct RFC
- Use for: complex UI/UX, cross-system orchestration, AI/ML, high-volume analytics

---

## Integration Patterns

**Core Rule**: Every integration must go through a released, stable, versioned API. Never call unreleased RFCs directly.

| Scenario | Pattern | Anti-Pattern |
|----------|---------|-------------|
| External system creates SAP doc real-time | Released OData POST API via CPI | Direct RFC call |
| React when SAP data changes | Business Event subscription via Event Mesh | Polling / QRFC |
| High-volume overnight sync | Released IDoc or bulk OData | Direct SQL extraction |
| Master data distribution | MDG + Business Events | Manual extracts |
| Complex cross-system orchestration | BTP CAP + Integration Suite | Custom middleware with C0 FMs |

**SAP Integration Suite components**: CPI (integration flows), API Management, Event Mesh, Integration Advisor (B2B/EDI), Open Connectors (170+ pre-built).

---

## Upgrade & Operations

### Why Clean Core Enables Continuous Innovation
- Dirty core: 12–18 months upgrade prep | Clean core: 2–4 weeks
- S/4HANA Public Cloud: quarterly updates | Private Cloud: 6-month | On-Premise: yearly
- C1 objects: SAP backward compatibility guaranteed | C0 objects: may break with no SAP obligation

### ABAP Test Cockpit (ATC) — Enforcement Gate
ATC runs static code analysis before transport. In mature governance: **ATC critical findings (Priority 1 / Level D) block transports to production**.

Checks include: C0 API usage, obsolete ABAP statements, direct SELECT on non-released tables, hard-coded system dependencies.

**Key SAP Notes**:
- Note 3578329: Framework Classification — which classic frameworks are B/C/D
- Note 3565942: ATC Check Assignment — which ATC checks map to each level

**S_ABPLNGVS**: Authorization object to restrict developers to 'ABAP for Cloud Development' language version — a powerful preventive governance control.

### Tools Reference

| Tool | Phase | Purpose |
|------|-------|---------|
| SAP Readiness Check | Assessment | One-time scan for S/4HANA upgrade readiness |
| Cloud ALM Custom Code Analytics | Assessment + Ongoing | Z-object A-D levels, ATC findings, dead code, upgrade risk |
| ABAP Test Cockpit (ATC) | Development + CI/CD | Blocks non-Clean Core from transport |
| SAP Business Accelerator Hub (api.sap.com) | Development | Catalog of all released OData APIs, Events, IDocs |
| SAP Integration Suite (BTP) | Integration | CPI, API Management, Event Mesh |
| SAP MDG | Data Governance | Master data governance hub |
| SAP Cloud ALM | Operations | Monitoring, alerting, test management, lifecycle |
| SCMON / SUSG | Ongoing | Identify Z-objects with zero runtime usage (dead code) |
| SMODILOG | Assessment | Log of SAP standard code modifications |
| SAP for Me — Clean Core Dashboard | Ongoing | Executive dashboard for RISE PCE systems |

---

## SAP Cloudification Repository (GitHub)

**Repo**: `https://github.com/SAP/abap-atc-cr-cv-s4hc`
**Web Viewer**: `https://sap.github.io/abap-atc-cr-cv-s4hc/`

The authoritative machine-readable source for released/unreleased API classification per system version. Contains released C1 objects, C0 objects with their C1 successors, classic APIs, and A-D level classifications.

### Pandora-Specific URLs (S/4HANA 2022 FPS02)

| Purpose | File | URL |
|---------|------|-----|
| **Released APIs (C0/C1) — Pandora version** | `objectReleaseInfo_PCE2022_2.json` | `https://raw.githubusercontent.com/SAP/abap-atc-cr-cv-s4hc/main/src/objectReleaseInfo_PCE2022_2.json` |
| Released APIs — always latest PCE | `objectReleaseInfo_PCELatest.json` | `https://raw.githubusercontent.com/SAP/abap-atc-cr-cv-s4hc/main/src/objectReleaseInfo_PCELatest.json` |
| **A-D level classifications (Note 3565942)** | `objectClassifications_SAP.json` | `https://raw.githubusercontent.com/SAP/abap-atc-cr-cv-s4hc/main/src/objectClassifications_SAP.json` |
| Classic APIs for Level B wrappers | `objectClassifications_3TierModel.json` | `https://raw.githubusercontent.com/SAP/abap-atc-cr-cv-s4hc/main/src/objectClassifications_3TierModel.json` |
| Offline CSV analysis | `objectReleaseInfo_PCE2022_2.csv` | same base URL, `.csv` extension |

> **Note**: JSON files exist per FPS release only — no per-SP-stack updates. FPS02 = suffix `_2`.

### Wiring into ATC Check Variant

1. In ABAP Test Cockpit, activate check: **Clean Core → "Usage of Released APIs (Cloudification Repository)"**
2. In check attributes, enter the Pandora-specific URL above (`objectReleaseInfo_PCE2022_2.json`)
3. For the new A-D checks (Note 3565942): use `objectClassifications_SAP.json` instead
4. SSL prerequisite: implement **Note 3582797** if you get SSL handshake failures from DS4 calling GitHub

### What the JSON Contains

- **Released (C1)** objects → safe to use in ABAP Cloud
- **Not Released (C0)** objects → with their C1 **successor** specified (e.g., `LFA1` → `I_Supplier`, `KONV` → released CDS successor)
- **Classic APIs** → Level B candidates for wrapper pattern
- **Deprecated** objects → with mandatory successor migration info

### Use in `ZTR_CLOUD_READINESS` Tooling

The JSON can be fetched programmatically from ABAP using `CL_HTTP_CLIENT` to look up any object name and determine:
- Its release status (C0/C1/C2)
- Its A-D level classification
- Its C1 successor (for remediation planning)

This avoids hardcoding the `ZTCC_CLOUD_API2` table — instead pulling live classification data directly from the SAP-maintained GitHub source.

---

## Governance & Strategy

### KPI Formulas

**Technical Debt Score (TDS)**:
```
TDS = (P1 findings × 10) + (P2 findings × 5) + (P3 findings × 1)
Target for upgrade readiness: TDS < 100
```

**Clean Core Share (CCS)**:
```
CCS = (Level A + Level B active Z-objects) ÷ (Total active Z-objects) × 100%
Target: > 80% for upgrade readiness; > 90% for mature organizations
```

### Target KPIs

| KPI | Target |
|-----|--------|
| % Z-objects using only C1 APIs | > 90% |
| ATC critical findings in production | 0 |
| Unreleased RFC calls in integration | 0 |
| % integrations using released OData/Events | > 85% |
| Dead code ratio | < 5% |
| Upgrade preparation time | < 30 days |
| ARB exception rate | < 20% |

### Three-Horizon Roadmap

| Horizon | Timeline | Key Activities |
|---------|----------|----------------|
| H1: Stop the Bleeding | 0–6 months | Freeze new non-Clean Core dev. Enable ATC enforcement. Retire dead Z-code. Establish governance board. Train devs. |
| H2: Remediate | 6–18 months | Refactor high-risk Z to ABAP Cloud. Replace RFC integrations. Implement MDG. Move complex to BTP. |
| H3: Sustain | 18–36 months | Clean Core KPIs as business reporting. Automated quality gates in CI/CD. Quarterly ARB reviews. |

### Three Lines of Defense

| Line | Role | Responsibilities |
|------|------|-----------------|
| Line 1: Dev Teams | Day-to-day | Follow Clean Core by design. ATC self-checks. |
| Line 2: Architecture Review Board (ARB) | Quality gate | Approve extensibility approach. Review CC score monthly. Exception management. |
| Line 3: Executive Sponsor | Strategic | Clean Core as KPI. Budget for remediation. Business case owner. |

---

## Advanced Topics (Whitepaper Deep-Dive)

For detailed coverage of these topics, read: `references/whitepaper-advanced.md`

- **Two-Layer IT Architecture** (S/4HANA stable core + BTP innovation layer)
- **SAP Application Extension Methodology (AEM)** — 3-phase decision framework
- **SAP Build + Joule AI** — low-code/no-code + AI co-pilot
- **Cloudification Repository** — authoritative C0/C1/C2 classification catalog
- **Stay Clean / Get Clean patterns** + **Boy Scout Principle**
- **Wrapper/Encapsulation strategy** for B/C remediation
- **Exemption Process** — formal exception governance with Quality Manager role
- **Process Category Stratification** (Standard / Enhanced / Custom / Innovative)
- **Governance Maturity Framework** (0–5 scale)
- **SAP LeanIX** (Enterprise Architecture) + **SAP Signavio** (Process Intelligence)
- **Solution Standardization Board (SSB)** vs. Architecture Review Board (ARB)
- **Partner Add-Ons + SAP ICC Certification**

---

## Brownfield vs. Greenfield

| Dimension | Greenfield | Brownfield |
|-----------|-----------|-----------|
| Priority | Enforce Clean Core from day 1 | Assess, classify, remediate |
| Quick win | 100% ABAP Cloud from start | Retire dead code (fast, low risk) |
| Biggest risk | Pressure to deviate 'just this once' | Running out of remediation budget |
| Timeline | Immediate | 2–4 year program |

**The Brownfield Trap**: 'Lift and shift' moves the problem, not solves it. True Clean Core brownfield requires courage to say: "We will not migrate this until it is clean."

---

## Key Statistics to Remember

- Typical S/4HANA brownfield: 2,000–5,000+ Z-objects
- 30–40% of Z-objects are dead code (never executed)
- ~60% of remaining Z-objects call unreleased (C0) APIs
- Dirty core upgrade prep: 12–18 months | Clean core: 2–4 weeks
- Architecture Challenge example: 3,800 → ~1,800 objects after 9-month program (-53%), upgrade time 22 months → 4–6 weeks (-95%)
