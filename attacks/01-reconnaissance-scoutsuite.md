# Reconnaissance — ScoutSuite AWS Security Audit

## Overview

ScoutSuite 5.14.0 was run against the target AWS account to simulate attacker
reconnaissance after gaining read-level credential access. ScoutSuite performs
read-only API enumeration across all AWS services and generates a risk report
identifying misconfigurations and attack surface.

## Attacker Mindset

At this stage of the kill chain, the attacker has obtained AWS credentials and
is mapping the environment before choosing an attack path. ScoutSuite replicates
exactly what a skilled attacker would do — enumerate everything readable, identify
weaknesses, prioritize the highest-value attack paths.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1526 | Cloud Service Discovery | Discovery |
| T1087.004 | Account Discovery: Cloud Account | Discovery |
| T1580 | Cloud Infrastructure Discovery | Discovery |

## Scan Details

| Field | Value |
|-------|-------|
| Tool | ScoutSuite 5.14.0 |
| Target | AWS account (details withheld) |
| Region | ap-south-1 (Mumbai) |
| Profile | default (Lalith IAM user) |
| Date | 2026-05-25 |
| Duration | ~5 minutes |

## Permission Errors Encountered

| Service | Error | Significance |
|---------|-------|-------------|
| S3 | s3:ListAllMyBuckets denied | DetectionLabPolicy does not grant bucket enumeration — attacker would note this gap |
| IAM | Failed to fetch user tags | Minor permission gap on tag enumeration |

## Findings Summary

| Service | Findings | Severity |
|---------|----------|---------|
| IAM | 4 | Medium |
| All other services | 0 | Clean |

## IAM Findings Detail

### Finding 1 — Minimum Password Length Too Short
- **Risk:** Weak passwords vulnerable to dictionary attacks against console login
- **Attack relevance:** Relevant for console-based attacks, not API programmatic access
- **Recommendation:** Set minimum length to 14+ characters

### Finding 2 — Password Expiration Disabled
- **Risk:** Compromised passwords remain valid indefinitely
- **Attack relevance:** If console credentials are obtained, attacker retains access permanently
- **Recommendation:** Enable password expiration with 90-day maximum

### Finding 3 — Password Policy Allows Reuse
- **Risk:** Users can cycle back to previously compromised passwords
- **Attack relevance:** Historical credential leaks remain viable attack vectors
- **Recommendation:** Enforce 24 password history minimum

### Finding 4 — Password Expiration Threshold Misconfiguration
- **Risk:** Conflicting password policy rules indicate incomplete policy configuration
- **Attack relevance:** Indicates policy was partially configured — signals immature security posture
- **Recommendation:** Review and consistently apply full password policy

## Attacker Conclusion

All 4 findings are password policy issues affecting console login security.
These do not affect programmatic API access which uses Access Key ID and
Secret Access Key — separate from console password policy entirely.

**Attack path decision:** Password policy findings are noted but do not alter
the primary attack path. Proceeding with programmatic access via Pacu for
IAM enumeration and privilege escalation attempts.

**Key distinction for detection:** ScoutSuite's read-only API calls generated
CloudTrail events but are indistinguishable from normal administrative activity.
This is the stealth advantage of reconnaissance using legitimate AWS API calls.

## Detection Gap Note

ScoutSuite made dozens of legitimate API calls during this scan. GuardDuty
did not alert on any of them. This demonstrates a fundamental detection gap:
**read-only enumeration using valid credentials is invisible to GuardDuty by default.**

This will be documented in Phase 4 Detection Gap Analysis.

## Next Step

Proceed to Pacu-based IAM enumeration for deeper privilege mapping and
identification of privilege escalation paths.