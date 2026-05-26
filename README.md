# Cloud Attack Simulation Lab

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![AWS](https://img.shields.io/badge/AWS-ap--south--1-orange)
![License](https://img.shields.io/badge/License-MIT-blue)

## Overview

A hands-on AWS cloud attack simulation lab demonstrating full kill chain execution,
detection gap analysis, and MITRE ATT&CK mapping. Built to develop and demonstrate
attacker mindset for cloud security engineering roles.

## Disclaimer

All simulations conducted on infrastructure owned by the author.
See [DISCLAIMER.md](DISCLAIMER.md) for full details.

## Project Status

| Phase | Title | Status |
|-------|-------|--------|
| Phase 1 | Environment Setup | Complete |
| Phase 2 | Reconnaissance and Enumeration | Complete |
| Phase 3 | Attack Simulation - Kill Chain | Complete |
| Phase 4 | Detection Gap Analysis | Complete |
| Phase 5 | MITRE ATT&CK Navigator Mapping | Complete |
| Phase 6 | Final Documentation and Report | Complete |

## Tools Used

| Tool | Purpose |
|------|---------|
| Pacu | AWS exploitation framework |
| ScoutSuite | Multi-cloud security auditing |
| AWS GuardDuty | Threat detection |
| AWS CloudTrail | API activity logging |
| MITRE ATT&CK Navigator | Technique visualization |

## MITRE ATT&CK Coverage

| Technique ID | Name | Tactic | Status |
|-------------|------|--------|--------|
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion | Complete |
| T1530 | Data from Cloud Storage | Collection | Complete |
| T1087.004 | Account Discovery: Cloud Account | Discovery | Complete |
| T1526 | Cloud Service Discovery | Discovery | Complete |
| T1580 | Cloud Infrastructure Discovery | Discovery | Complete |
| T1619 | Cloud Storage Object Discovery | Discovery | Complete |
| T1078.004 | Valid Accounts: Cloud Accounts | Defense Evasion | Complete |
| T1069.003 | Permission Groups Discovery | Discovery | Complete |

## Repository Structure

| Folder | Contents |
|--------|---------|
| attacks/ | Kill chain scripts and scenario documentation |
| detection/ | GuardDuty findings and CloudTrail evidence |
| reports/ | Detection gap analysis reports |
| tools/ | Tool configuration and notes |
| diagrams/ | Architecture diagrams |
| docs/ | Phase documentation |
| findings/screenshots/ | Blurred evidence screenshots |
| navigator/ | MITRE ATT&CK Navigator JSON |

## Author

Lalith
[GitHub](https://github.com/Lalith012)
