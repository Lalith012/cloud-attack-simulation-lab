# Attack Scenario 2 — Data Exfiltration: CloudTrail Log Theft

## Overview

This scenario simulates an attacker with read access to an S3 bucket
downloading CloudTrail logs to extract account intelligence. This is a
purely read-only attack — no data is modified or destroyed. The goal
is intelligence gathering to support more targeted subsequent attacks.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1530 | Data from Cloud Storage | Collection |
| T1119 | Automated Collection | Collection |
| T1087.004 | Account Discovery: Cloud Account | Discovery |

## Attack Narrative

An attacker with s3:GetObject permission and knowledge of the bucket name
downloads CloudTrail log files and extracts operational intelligence about
the target account — user identities, active hours, operating environment,
and service configuration.

## Pre-Attack Conditions

| Condition | Value |
|-----------|-------|
| Required Permission | s3:GetObject, s3:ListBucket |
| Bucket Name | Known from prior enumeration |
| CloudTrail Status | Active (read-only attack, no disruption needed) |

## Attack Execution

### Step 1 — List Available Log Files
Command: aws s3 ls s3://[bucket]/AWSLogs/[account]/CloudTrail/ap-south-1/2026/05/26/
Result: 16 log files from today's session visible

### Step 2 — Download Target Log File
Command: aws s3 cp s3://[bucket]/AWSLogs/[account]/CloudTrail/[target-file].json.gz
Result: 2,004 byte log file downloaded successfully

### Step 3 — Decompress and Parse
Method: PowerShell GZipStream decompression
Result: JSON log file readable — 3 CloudTrail events extracted

### Step 4 — Intelligence Extraction
Analyzed first 3 events from the downloaded log file

### Step 5 — Cleanup
Downloaded files removed from local machine after documentation

## Intelligence Extracted from 3 Log Events

### Identity Intelligence
| Field | Value |
|-------|-------|
| IAM User | Lalith |
| User Type | IAMUser |
| Account | Withheld |
| Access Key | AKIA************ |

### Operational Intelligence
| Field | Value |
|-------|-------|
| Source IP | 49.206.**.*** |
| Location | India |
| ISP | Fibernet |
| Operating System | Windows 11 |
| AWS CLI Version | 2.34.48 |
| Active Hours | ~10:47-10:49 UTC (16:17-16:19 IST) |

### Infrastructure Intelligence
| Finding | Value |
|---------|-------|
| CloudTrail trail name | detection-lab-trail |
| Active AWS services | Resource Explorer, CloudTrail, STS |
| Service roles | AWSServiceRoleForResourceExplorer active |

## Attacker Value of Extracted Intelligence

### Operational Timing Attack
Knowing the account owner is active 16:00-21:00 IST, an attacker
schedules their main attack for 02:00-06:00 IST — outside monitoring
hours when alerts are less likely to be investigated immediately.

### Targeted Disruption
Trail name detection-lab-trail confirmed — attacker knows exactly
which trail to disable in Scenario 1 without trial and error.

### Credential Context
Session tokens from AWS service role assumptions appear in CloudTrail
logs. Expired tokens have no value but active tokens could be reused
to impersonate service roles.

### Blending In
Knowing the normal user agent string and source IP allows an attacker
to spoof these values in subsequent API calls to blend in with
legitimate activity patterns.

## Detection Analysis

### What GuardDuty Detected
Nothing. S3 GetObject calls using valid credentials do not trigger
any GuardDuty findings by default.

### Detection Gap — S3 Read Access Is Invisible
GuardDuty does not alert on S3 object downloads using legitimate
credentials. An attacker can exfiltrate any readable S3 data
without triggering a single GuardDuty finding.

### What Would Detect This
- S3 Server Access Logging — logs every GetObject request
- AWS Macie — detects sensitive data access patterns in S3
- CloudTrail data events for S3 — logs individual object access
  (not enabled by default — management events only are default)

## Key Detection Gap Finding

CloudTrail data events for S3 are disabled by default. Only
management events are logged. This means S3 object downloads
do not appear in CloudTrail unless data event logging is
explicitly enabled.

This is documented as Detection Gap 2 in Phase 4.

## Azure Equivalent

| AWS | Azure |
|-----|-------|
| S3 CloudTrail logs | Azure Storage Account activity logs |
| AWS Macie | Microsoft Purview |
| S3 data event logging | Azure Storage diagnostic logging |
| GuardDuty S3 findings | Microsoft Defender for Storage |

## Status
Scenario 2 Complete — Data exfiltration demonstrated, intelligence
extraction documented, detection gaps identified.