# Phase 1 — Environment Setup

## Overview

This phase covers the complete setup of the Cloud Attack Simulation Lab environment,
including repository structure, tool installation, and pre-attack verification procedures.

## Environment Details

| Component | Details |
|-----------|---------|
| Cloud Provider | AWS |
| Region | ap-south-1 (Mumbai) |
| IAM User | Lalith (least privilege — DetectionLabPolicy) |
| CloudTrail | detection-lab-trail (active, logging verified) |
| GuardDuty | Enabled (detector verified) |
| Local OS | Windows 11 |
| Python | 3.x (virtual environments per tool) |

## Tools Installed

### Pacu 1.7.0
- **What it is:** Open-source AWS exploitation framework by Rhino Security Labs
- **Purpose:** Simulate real-world AWS attack scenarios — enumeration, privilege escalation, persistence
- **Install location:** D:\SecurityProjects\pacu-install\
- **Install method:** pip install pacu into isolated virtual environment
- **Authentication:** AWS credentials via environment variables (Pacu 1.7.0 Windows bug workaround)
- **Verification:** iam__detect_honeytokens module confirmed authenticated to correct ARN

### ScoutSuite 5.14.0
- **What it is:** Open-source multi-cloud security auditing tool by NCC Group
- **Purpose:** Read-only reconnaissance — maps attack surface across IAM, S3, EC2, CloudTrail
- **Install location:** D:\SecurityProjects\scoutsuite-install\
- **Install method:** pip install scoutsuite into isolated virtual environment
- **Verification:** scout --version confirmed 5.14.0

## Repository Structure

| Folder | Purpose |
|--------|---------|
| attacks/ | Kill chain scripts and scenario documentation |
| detection/ | GuardDuty findings and CloudTrail evidence |
| reports/ | Detection gap analysis reports |
| tools/ | Tool configuration and notes |
| diagrams/ | Architecture diagrams |
| docs/ | Phase documentation |
| findings/screenshots/ | Blurred evidence screenshots |
| navigator/ | MITRE ATT&CK Navigator JSON |

## Pre-Attack Verification Procedure

Before every session involving offensive tooling, the following must be verified:

1. aws sts get-caller-identity — confirm account 664858858896, user Lalith
2. aws cloudtrail get-trail-status — confirm IsLogging: true
3. aws guardduty list-detectors — confirm detector ID active
4. Pacu whoami after launch — confirm correct ARN before running any modules

## Installation Issues and Resolutions

| Issue | Cause | Resolution |
|-------|-------|-----------|
| readline module not found | Windows lacks Unix readline module | Installed pyreadline3 |
| pacu.core.models not found | Cloned repo folder conflicted with pip package name | Installed Pacu in separate directory outside cloned repo |
| models.py missing after install | Windows Defender quarantined file during install | Added Defender folder exclusion, force reinstalled |
| Credentials not persisting | Pacu 1.7.0 Windows bug with set_keys | Set credentials via PowerShell environment variables |

## Pacu Launch Procedure

cd D:\SecurityProjects\pacu-install
`$`env:AWS_ACCESS_KEY_ID="your_key"
`$`env:AWS_SECRET_ACCESS_KEY="your_secret"
`$`env:AWS_DEFAULT_REGION="ap-south-1"
venv\Scripts\activate
pacu

## ScoutSuite Launch Procedure

cd D:\SecurityProjects\scoutsuite-install
venv\Scripts\activate
scout aws --help

## Security Controls Applied

- Windows Defender exclusion scoped to SecurityProjects tool directories only
- AWS credentials never hardcoded — set via environment variables, cleared after session
- .gitignore excludes all credential files, Pacu session data, ScoutSuite reports
- DISCLAIMER.md present in repo root before any offensive tooling added
- All screenshots to be blurred before GitHub commit

## Status

Phase 1 Complete — Environment verified, tools installed, security controls applied.