# Attack Scenario 1 — Defense Evasion: CloudTrail Disruption

## Overview

This scenario simulates an attacker disabling AWS CloudTrail logging before
executing their main attack. Disabling logging eliminates the audit trail and
partially blinds GuardDuty's API-based detection capabilities.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion |

## Attack Narrative

An attacker with valid AWS credentials and cloudtrail:StopLogging permission
disables CloudTrail before executing their primary attack objective. This is
a precursor action — the goal is to operate without generating an audit trail.

## Pre-Attack State

| Field | Value |
|-------|-------|
| CloudTrail Status | IsLogging: true |
| Last Log Delivery | 2026-05-26T16:17:42 |
| GuardDuty Status | Active |
| Detection Coverage | Full API logging active |

## Attack Execution

### Step 1 — Confirm Baseline
Command: aws cloudtrail get-trail-status --name detection-lab-trail
Result: IsLogging: true — confirmed active before attack

### Step 2 — Stop CloudTrail Logging
Command: aws cloudtrail stop-logging --name detection-lab-trail
Result: No output — silence indicates success
Effect: CloudTrail immediately stopped delivering log files to S3

### Step 3 — Verify Attack Success
Command: aws cloudtrail get-trail-status --name detection-lab-trail
Result: IsLogging: false — confirmed logging stopped

### Step 4 — Restore Logging (Lab Safety)
Command: aws cloudtrail start-logging --name detection-lab-trail
Result: IsLogging: true — logging restored immediately

## GuardDuty Response

### Finding Generated
| Field | Value |
|-------|-------|
| Finding Type | Stealth:IAMUser/CloudTrailLoggingDisabled |
| Title | An AWS CloudTrail trail detection-lab-trail was disabled |
| Severity | 2.0 (Low) |
| Detection Time | ~10 minutes after attack |
| Count | 2 (May 20 setup + May 26 simulation) |

### Finding Description
"AWS CloudTrail trail detection-lab-trail was disabled by Lalith calling
StopLogging under unusual circumstances. This can be attackers attempt to
cover their tracks by eliminating any trace of activity performed while
they accessed your account."

## Detection Analysis

### What GuardDuty Detected
GuardDuty successfully identified the StopLogging API call and generated
a Stealth finding. Detection worked because CloudTrail logged the final
StopLogging API call before going silent — giving GuardDuty the event
it needed to generate a finding.

### Detection Gaps Identified

**Gap 1 — Severity Miscalibration**
GuardDuty assigns severity 2.0 (Low) to CloudTrail disabling. In a real
SOC, low severity findings are investigated during normal working hours
and may sit in a queue for hours or days. By that time the attacker has
already executed their main attack and restored logging.

Recommendation: Implement custom EventBridge rule to auto-escalate
Stealth:IAMUser/CloudTrailLoggingDisabled to Critical and trigger
immediate SNS notification regardless of GuardDuty severity score.

**Gap 2 — Alternative Disruption Methods Not Detected**
GuardDuty detected StopLogging because it generates a CloudTrail event.
Alternative disruption methods may evade this detection:
- Modifying S3 bucket policy to block log delivery — no GuardDuty finding
- Deleting the S3 bucket — no Stealth finding generated
- Modifying CloudTrail event selectors to exclude specific API calls

**Gap 3 — Window of Blindness**
Between StopLogging and detection (~10 minutes), all API activity is
unlogged. An attacker who moves quickly during this window leaves no
CloudTrail evidence.

## What GuardDuty Cannot Detect Without CloudTrail

| Detection Type | Without CloudTrail |
|---------------|-------------------|
| IAM enumeration | Not detected |
| Privilege escalation attempts | Not detected |
| Unauthorized API calls | Not detected |
| Credential abuse | Not detected |
| Network-based threats (VPC Flow Logs) | Still detected |
| DNS-based threats (Route53 logs) | Still detected |
| EC2 malware (Malware Protection) | Still detected |

## SOC Recommendation

CloudTrail disabling is a precursor attack technique — it signals that
a more serious attack is imminent. Default GuardDuty severity scoring
does not reflect this context.

Recommended response procedure:
1. Treat any Stealth:IAMUser/CloudTrailLoggingDisabled as Critical
2. Immediately re-enable CloudTrail
3. Check CloudTrail for the StopLogging event and identify the caller
4. Rotate credentials of the user who stopped logging
5. Review all API activity in the 15-minute window before StopLogging

## Azure Equivalent

| AWS | Azure |
|-----|-------|
| CloudTrail StopLogging | Disabling Azure Monitor Diagnostic Settings |
| GuardDuty Stealth finding | Microsoft Defender for Cloud alert |
| EventBridge escalation rule | Azure Monitor Alert Rule |

## Status
Scenario 1 Complete — Defense evasion demonstrated, GuardDuty finding
captured, detection gaps documented.