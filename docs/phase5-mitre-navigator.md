# Phase 5 — MITRE ATT&CK Navigator Mapping

## Overview

This phase covers the creation of a MITRE ATT&CK Navigator layer file
mapping all techniques executed in Project 2 to the ATT&CK Enterprise
matrix. The Navigator layer provides a visual heatmap of technique
coverage for use in security team communications and portfolio presentation.

## Navigator File Location

Layer file: navigator/attack-navigator-layer.json
Screenshot: findings/screenshots/navigator-heatmap.png

## Techniques Mapped

| Technique ID | Name | Tactic | Status |
|-------------|------|--------|--------|
| T1526 | Cloud Service Discovery | Discovery | Executed |
| T1087.004 | Account Discovery: Cloud Account | Discovery | Executed |
| T1580 | Cloud Infrastructure Discovery | Discovery | Executed |
| T1530 | Data from Cloud Storage | Collection | Executed |
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion | Executed |
| T1619 | Cloud Storage Object Discovery | Discovery | Executed |
| T1078.004 | Valid Accounts: Cloud Accounts | Defense Evasion | Simulated |
| T1069.003 | Permission Groups Discovery: Cloud Groups | Discovery | Executed |

## Color Legend

| Color | Meaning |
|-------|---------|
| Red (#ff6666) | Fully executed in lab environment |
| Orange (#ffaa00) | Simulated — technique represented but not fully executed |

## ATT&CK Version

Layer created in ATT&CK v14, upgraded to v19 on load.
Navigator file updated to reflect v19.

## How to Load the Navigator Layer

1. Go to https://mitre-attack.github.io/attack-navigator/
2. Click Open Existing Layer
3. Click Upload from local
4. Select navigator/attack-navigator-layer.json
5. Click Yes to upgrade to latest ATT&CK version if prompted

## Interview Relevance

The Navigator layer demonstrates:
- Systematic thinking about technique coverage
- Ability to communicate security findings visually
- Familiarity with the ATT&CK framework at a practical level
- Understanding of which tactics and techniques apply to cloud environments

## Cross-Project ATT&CK Coverage

Combined with Project 1, the portfolio now covers:

| Tactic | Project 1 | Project 2 |
|--------|-----------|-----------|
| Initial Access | T1078.004 | T1078.004 |
| Discovery | T1087.004 | T1526, T1087.004, T1580, T1619 |
| Collection | T1530 | T1530 |
| Defense Evasion | T1562.008 | T1562.008, T1078.004 |
| Execution | — | — |
| Persistence | — | — |

## Status
Phase 5 Complete — MITRE ATT&CK Navigator layer created, uploaded,
verified, and screenshot captured.