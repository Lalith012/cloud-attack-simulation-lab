# Phase 4 — Detection Gap Analysis

## Overview

This phase documents the formal detection gap analysis performed after
all three attack scenarios in Phase 3. It identifies what GuardDuty
detected, what it missed, why each gap exists, and specific remediation
recommendations for each gap.

## Full Report Location

Complete detection gap analysis: reports/detection-gap-analysis.md

## Summary of Gaps Identified

| Gap | Scenario | Description | Severity |
|-----|----------|-------------|---------|
| 1 | 1 | GuardDuty severity miscalibration — StopLogging rated Low | High |
| 2 | 1 | Alternative CloudTrail disruption methods not detected | High |
| 3 | 1 | GuardDuty detection lag — 10 minute window | Medium |
| 4 | 2 | S3 read access invisible to GuardDuty | High |
| 5 | 2 | CloudTrail data events disabled by default | High |
| 6 | 3 | Silence window contents unrecoverable | Critical |
| 7 | 3 | Finding aggregation reduces urgency perception | Medium |

## Detection Coverage

| Metric | Value |
|--------|-------|
| Total attack actions | 7 |
| Detected by GuardDuty | 2 |
| Missed entirely | 5 |
| Default detection rate | 28% |
| Estimated rate after remediation | 85% |

## Key Recommendations

### Immediate Priority
1. EventBridge rule to auto-escalate Stealth:IAMUser/CloudTrailLoggingDisabled
2. Enable CloudTrail data events on CloudTrail log bucket
3. Enable GuardDuty S3 Protection feature

### Medium Priority
4. AWS Config rule for S3 bucket policy changes on log bucket
5. Real-time CloudTrail to Lambda pipeline for instant StopLogging detection
6. Enable AWS Macie on CloudTrail log bucket

### Long Term
7. VPC Flow Logs for network visibility during CloudTrail gaps
8. Endpoint detection on EC2 instances independent of CloudTrail

## Most Critical Finding

GuardDuty detected only 28% of attack actions with default configuration.
The most dangerous actions — enumeration under silence and S3 data
exfiltration — were completely invisible. This demonstrates that GuardDuty
alone is insufficient for comprehensive cloud threat detection without
additional configuration and complementary controls.

## MITRE ATT&CK Coverage

| Technique ID | Detected | Detection Method |
|-------------|---------|-----------------|
| T1562.008 | Partial | GuardDuty — low severity, delayed |
| T1530 | No | Not detected by default |
| T1087.004 | No | Not detected under silence |
| T1580 | No | Not detected |
| T1619 | No | Not detected |

## Status
Phase 4 Complete — Detection gaps identified, quantified, and
remediation recommendations documented.