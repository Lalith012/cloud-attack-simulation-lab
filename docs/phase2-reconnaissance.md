# Phase 2 — Reconnaissance and Enumeration

## Overview

This phase covers the reconnaissance and enumeration activities performed
against the target AWS account using ScoutSuite and Pacu. The goal was to
map the attack surface, identify misconfigurations, and determine viable
attack paths before executing simulated attacks in Phase 3.

## Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| ScoutSuite | 5.14.0 | Multi-cloud security auditing and attack surface mapping |
| Pacu | 1.7.0 | AWS exploitation framework for IAM enumeration |
| AWS CLI | 2.34.48 | Direct API enumeration for IAM and S3 |

## Reconnaissance Activities

### 1. ScoutSuite Account Audit
- Full AWS account audit performed using read-only API calls
- 24 services enumerated across the account
- 4 IAM findings identified — all password policy issues
- S3 bucket enumeration blocked by missing s3:ListAllMyBuckets permission
- Full findings documented in attacks/01-reconnaissance-scoutsuite.md

### 2. Pacu IAM Enumeration
- 1 IAM user enumerated — Lalith
- 6 IAM roles enumerated — all service-linked roles
- 1 custom policy enumerated — DetectionLabPolicy
- 0 IAM groups found
- Full analysis documented in attacks/02-iam-enumeration.md

### 3. S3 Enumeration
- Direct bucket access confirmed using known bucket name
- CloudTrail log files and digest files enumerated
- Bucket policy retrieved and analyzed — correctly configured
- Critical attack path identified via s3:PutBucketPolicy permission
- Full findings documented in attacks/03-s3-enumeration.md

## Key Findings

### Attack Surface Identified
| Finding | Risk | Details |
|---------|------|---------|
| cloudtrail:StopLogging permitted | CRITICAL | Attacker can disable audit logging |
| s3:GetObject permitted | HIGH | CloudTrail logs readable by attacker |
| s3:PutBucketPolicy permitted | HIGH | Bucket policy can be overwritten |
| No assumable roles | LOW | No role-based privilege escalation path |
| Password policy weak | MEDIUM | Console login vulnerable to dictionary attacks |

### Attack Paths Identified
1. CloudTrail disruption via StopLogging
2. S3 data exfiltration via GetObject
3. Recon under silence — chain of 1 and 2

## Detection Observations

All reconnaissance activities used legitimate AWS API calls. GuardDuty
generated zero findings during Phase 2. Read-only enumeration using valid
credentials is invisible to GuardDuty by default.

## MITRE ATT&CK Coverage

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1526 | Cloud Service Discovery | Discovery |
| T1087.004 | Account Discovery: Cloud Account | Discovery |
| T1580 | Cloud Infrastructure Discovery | Discovery |
| T1619 | Cloud Storage Object Discovery | Discovery |
| T1069.003 | Permission Groups Discovery: Cloud Groups | Discovery |

## Status
Phase 2 Complete — Attack surface mapped, viable attack paths identified,
all findings documented.