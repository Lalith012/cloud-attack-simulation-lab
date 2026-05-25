# S3 Enumeration — Bucket Discovery and Policy Analysis

## Overview

Following IAM enumeration, S3 was targeted as the next reconnaissance objective.
The target bucket stores CloudTrail logs — making it a high-value target for
both intelligence gathering and evidence destruction.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1530 | Data from Cloud Storage | Collection |
| T1619 | Cloud Storage Object Discovery | Discovery |
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion |

## Tools Used

| Tool | Command | Purpose |
|------|---------|---------|
| AWS CLI | s3 ls | Bucket content enumeration |
| AWS CLI | s3api get-bucket-policy | Bucket policy retrieval |
| ScoutSuite | AWS scan | Initial bucket discovery attempt |

## Bucket Discovery

### Initial Attempt — ListAllMyBuckets
ScoutSuite and `aws s3 ls` both failed with AccessDenied on
`s3:ListAllMyBuckets`. DetectionLabPolicy does not grant bucket
discovery permissions — only access to known buckets.

**Attacker note:** Without bucket discovery permissions, an attacker
must know bucket names in advance or use enumeration techniques such
as guessing common naming patterns or reading bucket names from
CloudTrail logs and other sources.

### Successful Enumeration — Known Bucket
Direct access to known bucket succeeded using `s3:ListBucket`:

| Field | Value |
|-------|-------|
| Bucket | detection-lab-cloudtrail-logs-[account-id] |
| Region | ap-south-1 |
| Contents | CloudTrail logs and digest files |
| Access | Read permitted via DetectionLabPolicy |

## Bucket Contents Analysis

### CloudTrail Log Files
- Location: AWSLogs/[account-id]/CloudTrail/
- Format: Gzip compressed JSON (.json.gz)
- Coverage: Multi-region — ap-south-1, ap-northeast-1, ap-northeast-2, ap-northeast-3
- Significance: Complete API activity history for the account

### CloudTrail Digest Files
- Location: AWSLogs/[account-id]/CloudTrail-Digest/
- Format: Digitally signed JSON (.json.gz)
- Significance: Integrity verification chain — detects log tampering

**Attacker implication:** Digest files mean deleting log files alone
is insufficient to cover tracks. A forensic investigator can use
digest files to detect gaps in the log chain. Stopping CloudTrail
before attacking is more effective than deleting logs after.

## Reconnaissance Value of CloudTrail Logs

CloudTrail logs accessible to the attacker contain:
- Complete API call history — every action ever taken in the account
- Source IP addresses of all API calls
- User agent strings revealing tools used
- Timestamps enabling activity pattern analysis
- Resource ARNs of all accessed services

An attacker reading these logs gains intelligence about:
- Normal activity patterns to blend in with
- Other services and resources in the account
- Historical access patterns and credential usage

## Bucket Policy Analysis

The bucket policy allows only two principals:

| Principal | Action | Purpose |
|-----------|--------|---------|
| cloudtrail.amazonaws.com | s3:GetBucketAcl | CloudTrail access verification |
| cloudtrail.amazonaws.com | s3:PutObject | CloudTrail log delivery |

**Finding:** Bucket policy is correctly configured. No public access,
no misconfigured principals, no excessive permissions.

## Critical Attack Path Identified

Although the bucket policy is secure, DetectionLabPolicy grants
`s3:PutBucketPolicy` to the Lalith user. This means an attacker
with these credentials can overwrite the bucket policy entirely.

**Attack path:**
1. Stop CloudTrail logging via cloudtrail:StopLogging
2. Overwrite bucket policy via s3:PutBucketPolicy to block log delivery
3. Execute main attack with no logging active
4. Optionally delete existing logs via s3:DeleteObject
5. Restore bucket policy to original to hide tampering

This attack path is documented as Attack Scenario 1 in Phase 3.

## Detection Gap Identified

All S3 enumeration commands generated CloudTrail events. However
GuardDuty did not alert on any reconnaissance activity. Read-only
enumeration using valid credentials remains invisible to GuardDuty.

This is documented as Detection Gap 1 in Phase 4.

## Summary

| Finding | Risk | Notes |
|---------|------|-------|
| CloudTrail logs readable | HIGH | Complete API history accessible to attacker |
| Bucket policy correctly configured | LOW | No misconfiguration found |
| s3:PutBucketPolicy granted to user | CRITICAL | Attacker can overwrite bucket policy |
| s3:DeleteObject granted to user | HIGH | Evidence destruction possible |
| Multi-region logging active | DEFENSIVE | Cannot evade logging by switching regions |