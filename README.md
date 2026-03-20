# SAP Clean Core — Claude Skill

A comprehensive Claude skill for SAP Clean Core architecture, extensibility, governance, and remediation.

## What this skill covers

- **A-D Extensibility Levels** (August 2025 model) and C0/C1/C2 release contracts
- **Quick decision framework** — Key User → BAdI → RAP → BTP
- **Cloudification Repository** integration with version-specific JSON URLs
- **Pandora-specific**: S/4HANA 2022 FPS02 (`objectReleaseInfo_PCE2022_2.json`)
- ATC enforcement, KPI formulas (TDS, CCS), governance boards (ARB, SSB)
- Integration patterns (OData, Business Events, RFC replacement)
- Upgrade readiness playbook and Three-Horizon roadmap

## Files

| File | Purpose |
|------|---------|
| `sap-clean-core/SKILL.md` | Main skill — loaded on every trigger |
| `sap-clean-core/references/whitepaper-advanced.md` | Deep-dive reference — loaded on demand |
| `sap-clean-core.skill` | Pre-packaged installable skill file |

## Installation

Download `sap-clean-core.skill` and install it via the Claude Skills interface.

## Key References

- [SAP Cloudification Repository](https://github.com/SAP/abap-atc-cr-cv-s4hc)
- [Cloudification Repository Viewer](https://sap.github.io/abap-atc-cr-cv-s4hc/)
- [SAP Business Accelerator Hub](https://api.sap.com)
- SAP Notes: 3565942 (ATC checks), 3578329 (framework classification), 3582797 (SSL setup)
