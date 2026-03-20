# SAP Clean Core — Whitepaper Advanced Reference

This file contains detailed coverage of advanced topics from the SAP Official 'Clean Core Extensibility' White Paper. Load this when the user needs deep-dive information beyond the SKILL.md main reference.

---

## Two-Layer IT Architecture

| Layer | Name | Description | Key Technologies |
|-------|------|-------------|-----------------|
| Layer 1 | Stable Core (S/4HANA) | Standardized, upgrade-safe ERP. Minimal customization — only SAP-released extension points used. | S/4HANA Cloud, Key User Extensibility, ABAP Cloud BAdIs, Released OData APIs |
| Layer 2 | Innovation Layer (BTP) | Agile, composable layer for complex, differentiated, experimental capabilities. | SAP BTP CAP, SAP Build, SAP Integration Suite, SAP AI Core, SAP Datasphere |

Every extensibility decision: "Does this belong in Layer 1 (use a released API) or Layer 2 (build on BTP)?"

---

## SAP Application Extension Methodology (AEM)

Three-phase decision framework — channel every extension request through this before any development begins:

| Phase | Name | Focus | Key Questions |
|-------|------|-------|--------------|
| Phase 1 | Assess the Use Case | Determine what you need and whether it's truly custom | Is this covered by standard SAP? Is a business process change possible? What is the business value of customizing? |
| Phase 2 | Assess the Technology | Select the right extensibility layer and tool | What extension point exists? What is the effort/risk? Which layer (Key User / ABAP Cloud / BTP) fits? |
| Phase 3 | Define the Target Architecture | Design the extension formally and get ARB approval | How will this be maintained? How does it affect upgrade? What is the remediation plan if the API changes? |

---

## SAP Build — Unified Development Platform (BTP Layer 2)

- **SAP Build Apps**: Visual application builder for web/mobile — connects via OData APIs
- **SAP Build Process Automation**: Workflow and RPA for process extensions
- **SAP Build Work Zone**: Digital workspace / portal for consolidated business user experiences
- **SAP Build Code**: AI-assisted environment for professional developers building CAP extensions

### Joule — AI Co-Pilot
- In SAP Build Code: Generates ABAP Cloud and CAP code snippets
- In S/4HANA: Triggers business processes via natural language — only through released APIs
- In SAP Build Process Automation: Builds automation flows from natural language
- **Clean Core relevance**: Because Joule only surfaces released APIs, AI-driven innovations only work on a Clean Core foundation. A dirty core cannot benefit from Joule's capabilities.

---

## Cloudification Repository

SAP-maintained catalog classifying every ABAP object by cloud readiness. Authoritative source for deciding if an object is safe.

| Classification | Meaning | Action |
|---------------|---------|--------|
| Released (C1) | Formally released, stable across upgrades | Safe — preferred choice |
| Released for Key Users (C2) | Key user config-level only | Only in Key User Extensibility context |
| Not Released (C0) | Internal SAP object — not for customer use | Never use — ATC blocks this |
| Deprecated | Superseded — successor exists | Migrate to successor immediately |
| To Be Deleted | Will be removed in future release | Immediate mandatory migration |

Access via: SAP Business Accelerator Hub (api.sap.com) or SAP Note 3578329.

---

## ATC Finding Priority Levels — Clean Core Mapping

| Clean Core Level | ATC Priority | Severity | Transport Behavior | Action Required |
|-----------------|-------------|---------|-------------------|----------------|
| Level A | No finding | Clean | Always passes | None — target state |
| Level B | Priority 3 | Information | Passes — informational | Log and monitor; migration backlog |
| Level C | Priority 2 | Warning | Passes by default; can be configured to block | Time-boxed remediation plan required |
| Level D | Priority 1 | Error | **BLOCKS transport by default** | Immediate remediation — zero tolerance |

**SAP Note 3565942**: Documents specific ATC checks assigned to each level.
**S_ABPLNGVS**: Authorization object — restrict to 'ABAP for Cloud Development' to prevent non-Clean Core code at IDE level.

---

## KPI Formulas (Detailed)

### Technical Debt Score (TDS)
```
TDS = (Count of Priority 1 ATC findings × 10) + (Count of Priority 2 findings × 5) + (Count of Priority 3 findings × 1)

Example: 50 P1 + 200 P2 + 500 P3 = 500 + 1000 + 500 = TDS of 2,000
Target for upgrade readiness: TDS < 100 (fewer than 10 P1 errors with documented plan)
```

### Clean Core Share (CCS)
```
CCS = (Count of Z-objects at Level A + Level B) ÷ (Total ACTIVE Z-objects) × 100%
- Only ACTIVE objects counted (runtime usage in past 12 months)
- Dead code excluded from denominator after retirement
- Target: > 80% for upgrade readiness; > 90% for mature organizations
```

### Additional KPI Tools

| KPI | Transaction / Tool | What It Measures |
|-----|--------------------|-----------------|
| Unused Code Share | SCMON / SUSG | % Z-objects with zero runtime usage in past 12 months |
| Business Modifications | SMODILOG | Count and nature of SAP standard code modifications |
| Custom Code Usage | Cloud ALM Custom Code Analytics | Usage frequency, calling programs, business criticality |
| ATC Finding Trend | ATC + Cloud ALM | Week-over-week change — improving or accumulating new debt? |

---

## Stay Clean / Get Clean Patterns

| STAY CLEAN (Preventive) | GET CLEAN (Remediation) |
|------------------------|------------------------|
| ATC enforcement gate in CI/CD pipeline | Full ATC scan to baseline current state |
| Architecture Review Board for all custom dev | Prioritized remediation backlog in Cloud ALM |
| S_ABPLNGVS restricts language version | Boy Scout Principle for incremental improvement |
| Fit-to-Standard workshops before every project | Retire dead code first (highest ROI) |
| Solution Standardization Board reviews | Wrapper/encapsulation strategy for B/C objects |
| Developer training on ABAP Cloud mandatory | Monthly KPI reporting against TDS and CCS |

### The Boy Scout Principle
**Rule**: Every time a developer touches an existing Z-object, they must leave it in a better Clean Core state than they found it.

**Practice**: Before modifying a B-level or C-level object, first migrate it to Level A compliance. Then make the functional change.

**Impact**: Over 12–18 months of active development, this incrementally cleans the portfolio without requiring a dedicated remediation sprint.

**Tracking**: Pre-modification ATC state recorded; post-modification ATC findings must be equal or fewer.

---

## Wrapper/Encapsulation Strategy for B/C Remediation

For B/C-level objects that can't be fully refactored immediately:

1. **Identify** the B/C-level dependency (e.g., Z-FM calling C0 API)
2. **Create** an ABAP Cloud-compliant wrapper class (ABAP Cloud language version) encapsulating the call
3. **Add** an exemption comment block: non-compliant API used, business justification, owner, planned remediation date
4. **All consuming Z-objects** call the wrapper class instead of the problematic API directly
5. **Register** the wrapper in Cloud ALM exemption list with time-bound remediation plan

**Result**: Converts scattered C-level usage into controlled, documented, time-boxed exceptions. When SAP releases the successor API, only the wrapper needs to change.

---

## Formal Exemption Process

| Step | Activity | Owner | Time Limit |
|------|----------|-------|-----------|
| 1. Request | Developer submits with business justification, risk assessment, proposed remediation date | Developer + Team Lead | At time of ATC finding |
| 2. Review | Quality Manager reviews against Clean Core policy | Quality Manager / ARB | Within 5 business days |
| 3. Registration | Approved exemptions registered in Cloud ALM with: object name, level, justification, owner, expiry date | Quality Manager | Immediately after approval |
| 4. Monitoring | Exemption list reviewed quarterly; 10–20% must be closed each cycle | Clean Core Program Lead | Quarterly |
| 5. Expiry | On expiry: renew with escalation justification OR complete remediation | Developer + Program Lead | Per agreed date |

**Quality Manager**: Formal SAP-recommended governance role responsible for approving exemptions and maintaining the exemption register. In smaller orgs, this may be the Technical or Solution Architect.

---

## Process Category Stratification

| Process Layer | Definition | Examples | Clean Core Guidance |
|--------------|-----------|---------|---------------------|
| Standard | Commodity processes — no differentiation needed | Payroll, Basic GL posting, Standard procurement | Run 100% standard. Zero customization. |
| Enhanced | Standard + minor additions for local compliance or efficiency | Local tax fields, Country-specific reporting, Plant-specific workflows | Key User Extensibility or released BAdIs only. Never modify standard. |
| Custom | Operational advantage — company-specific but not differentiating vs. competition | Internal approval hierarchies, Company-specific pricing, Internal reporting | ABAP Cloud on-stack or BTP CAP. Released APIs only. |
| Innovative | True competitive differentiators — unique to the company's business model | Proprietary supply chain optimization, AI-driven demand sensing | BTP side-by-side. Full innovation freedom in Layer 2. |

**Key insight**: 60–70% of 'custom' processes actually fall into Standard or Enhanced — meaning they can be delivered with zero or minimal customization. This is the business case for Fit-to-Standard workshops.

---

## Governance Maturity Framework (0–5 Scale)

| Maturity Level | Label | Characteristics |
|---------------|-------|----------------|
| 0 — Initial | Ad-hoc | No Clean Core policies. Developers use any API. No ATC enforcement. Upgrades painful and unpredictable. |
| 1 — Managed | Aware | Clean Core policy exists on paper. ATC installed but not enforced. Some developers aware. No exemption process. |
| 2 — Defined | Governed | ATC enforces Priority 1 blocking. ARB established. Exemption process defined. AEM used for new extensions. |
| 3 — Quantified | Measured | TDS and CCS tracked monthly. Dead code retired. Remediation backlog prioritized. KPIs reported to leadership. |
| 4 — Optimized | Improving | Boy Scout Principle in culture. All new dev Level A. Exemptions < 10%. Upgrade prep < 30 days. |
| 5 — Leading | Innovating | Zero Level D. CCS > 95%. Full BTP Layer 2 strategy. Quarterly SAP innovations adopted within 1 quarter of release. |

**Recommendation**: Target Maturity Level 3 before committing to RISE PCE upgrade. Target Level 4 within 24 months of go-live.

---

## SAP LeanIX & SAP Signavio

### SAP LeanIX — Enterprise Architecture Management
- **Application Landscape**: Maps business capabilities to applications
- **Obsolescence Management**: Identifies legacy extensions replaceable by new SAP standard features
- **Integration Landscape**: Documents all integration flows — prioritizes RFC-to-OData migration
- **Clean Core Reporting**: Connects Cloud ALM data to show business capability impact of technical debt
- **When to use**: Essential for complex landscapes (>500 Z-objects, multiple systems, many integrations)

### SAP Signavio — Business Process Intelligence
- **Process Discovery**: Mines actual process variants from S/4HANA transaction logs
- **Conformance Checking**: Identifies where actual processes deviate from SAP best-practice
- **Process Standardization**: Shows which custom extensions support non-standard process variants → "Fix the process, not the system"
- **When to use**: In Fit-to-Standard workshops and before major brownfield upgrade programs

---

## Architecture Review Board (ARB) vs. Solution Standardization Board (SSB)

| Dimension | Architecture Review Board (ARB) | Solution Standardization Board (SSB) |
|-----------|--------------------------------|--------------------------------------|
| Focus | Technical: Which APIs? Which extension pattern? ATC compliance? | Business: Which processes are standard? Where are we deviating from best practice? |
| Members | SAP Technical Architects, Lead Developers, Clean Core Program Lead | Business Process Owners, Solution Architects, Finance/HR/Supply Chain leaders |
| Trigger | Any new custom development request | Any new project, business requirement, or process redesign |
| Output | Technical design approval or rejection | Process classification: Standard / Enhanced / Custom / Innovative |
| Meeting Cadence | Weekly (active project), Monthly (steady state) | Per project initiation, then quarterly |

**Why SSB matters**: Without SSB, business owners can bypass ARB by framing technical requirements as mandatory business needs. SSB ensures business stakeholders own the 'Standard vs. Custom' decision first.

---

## Partner Add-Ons and SAP ICC Certification

**SAP ICC (Integration and Certification Center)**: Formal certification for third-party partner add-ons on S/4HANA.

Clean Core Certification Requirements for Partners:
- Only C1 released APIs — no C0 API usage permitted
- Zero Priority 1 (Level D) ATC findings
- No SAP standard code modifications
- Integration points via released OData APIs or Business Events only
- Upgrade-tested against latest S/4HANA release

**Customer action**: Always request SAP ICC certification status before procuring partner add-ons. Check: SAP Business Accelerator Hub and the SAP Certified Solutions Directory.

---

## Key SAP Notes & References

| Resource | Topic | Use When |
|----------|-------|---------|
| SAP Note 3578329 | Framework Classification — which classic ABAP frameworks are A/B/C/D | Evaluating whether a specific classic framework (user exit, enhancement, BAdI type) is acceptable at Level B or must be remediated |
| SAP Note 3565942 | ATC Check Assignment — which ATC checks correspond to each Clean Core level | Configuring your ATC variant to enforce the right checks for your target Clean Core level |
| api.sap.com (Business Accelerator Hub) | Catalog of all released SAP APIs, Business Events, and IDocs | Finding the Clean Core replacement for any RFC or legacy integration endpoint |
| SAP for Me — Clean Core Dashboard | Real-time compliance view for RISE PCE systems showing A-D distribution | Monthly leadership reporting and ongoing compliance monitoring |
| ABAP Platform — What's New | Quarterly updates: new released APIs, new BAdIs, new ABAP Cloud capabilities | Identifying new opportunities to replace B/C/D extensions with Level A alternatives |

---

## Architecture Challenge — Real-World Transformation Example

### Scenario
- 3,800 Z-objects | 140 RFC-based integrations | No MDG | 3 satellite ERP systems
- SAP Readiness Check: 2,200 Z-objects calling C0 APIs | 90 integrations using unreleased RFCs
- Estimated upgrade remediation at current state: **22 months**
- Mandate: RISE PCE upgrade in **9 months**

### Results After 9-Month Program

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Total Z-objects | 3,800 | ~1,800 | -53% |
| Level D objects | ~400 | 0 | -100% |
| Level C objects | ~1,800 | ~200 (documented) | -89% |
| Level A/B objects | ~300 | ~1,600 | +433% |
| RFC integrations | 140 | ~15 (legacy only) | -89% |
| OData/Event integrations | 0 | ~125 | New capability |
| Estimated upgrade time | 22 months | **4–6 weeks** | -95% |
| Clean Core Score | ~22% | ~85% | +63 points |

### Key Phases
1. **Phase 0** (Weeks 1–3): Rapid assessment — Cloud ALM Custom Code Analytics, ATC mass check, SUSG dead code analysis, RFC-to-OData mapping. Expected: ~1,200–1,500 dead code objects, ~600 RFCs with direct OData equivalents.
2. **Phase 1** (Weeks 1–8): Governance + quick wins — ARB established, ABAP Cloud profile enabled, Level D ATC blocking configured, dead code retired (~1,300 objects), ABAP Cloud training launched.
3. **Phase 2** (Weeks 4–16): Integration sprint — top 30 highest-risk RFCs prioritized, CPI flows built, event-driven patterns for WMS/MES. Target: 90 of 140 integrations remediated.
4. **Phase 3** (Weeks 8–28): ABAP Cloud refactoring — ~400–600 active complex Z-objects (after dead code + standard replacements), classified by criticality, high-risk objects fully refactored to RAP/ABAP Cloud.
5. **Phase 4** (Weeks 26–36): Upgrade readiness verification — re-run SAP Readiness Check, full ATC check, upgrade simulation in sandbox.
