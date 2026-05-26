# Attack Scenario 3 — Reconnaissance Under Silence

## Overview

This scenario chains defense evasion with active reconnaissance — stopping
CloudTrail logging, performing enumeration with no audit trail, then
restarting logging to conceal the gap. This is the most realistic attack
scenario as it demonstrates coordinated technique chaining.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion |
| T1087.004 | Account Discovery: Cloud Account | Discovery |
| T1580 | Cloud Infrastructure Discovery | Discovery |

## Attack Narrative

An attacker with valid credentials stops CloudTrail logging, performs
IAM and S3 enumeration during the silence window, then restarts logging
to hide the gap. Actions taken during the silence window leave no
CloudTrail evidence and are unrecoverable from logs alone.

## Attack Execution

### Step 1 — Stop CloudTrail Logging
Command: aws cloudtrail stop-logging --name detection-lab-trail
Verification: IsLogging: false confirmed

### Step 2 — IAM Enumeration Under Silence
Command: aws iam list-users
Result: 1 user enumerated — Lalith IAMUser
CloudTrail Event Generated: None — logging was disabled

### Step 3 — S3 Enumeration Under Silence
Command: aws s3 ls s3://[bucket-name]
Result: AWSLogs/ folder confirmed accessible
CloudTrail Event Generated: None — logging was disabled

### Step 4 — Restart CloudTrail Logging
Command: aws cloudtrail start-logging --name detection-lab-trail
Verification: IsLogging: true confirmed

### Step 5 — Verify Silence Gap
Command: aws cloudtrail lookup-events --lookup-attributes
AttributeKey=EventName,AttributeValue=StopLogging

## Evidence of Silence Gap

CloudTrail recorded the StopLogging event at 16:20:18 IST on 2026-05-26.
The IAM enumeration and S3 enumeration performed after that point do not
appear anywhere in CloudTrail. They happened inside the silence window
and left no audit trail.

| Event | Logged in CloudTrail |
|-------|---------------------|
| StopLogging | Yes — last event before silence |
| aws iam list-users | No — silence window |
| aws s3 ls | No — silence window |
| StartLogging | Yes — first event after silence |

## What a Forensic Investigator Sees

A forensic investigator reviewing CloudTrail would observe:
- StopLogging event at 16:20:18 IST
- StartLogging event shortly after
- A gap with zero API events in between
- No evidence of what actions were taken during the gap

The gap itself is suspicious and detectable. What happened inside
the gap is unrecoverable from CloudTrail alone.

## GuardDuty Response

### Findings Generated
GuardDuty finding count remained at 3 during and after Scenario 3.
The Stealth:IAMUser/CloudTrailLoggingDisabled finding from Scenario 1
was updated with an incremented count rather than a new finding.

### Detection Lag Demonstrated
GuardDuty did not immediately generate a new finding for the Scenario 3
StopLogging event. This lag window — estimated 5-15 minutes — represents
additional time during which an attacker can operate before any alert
reaches a SOC analyst.

## Detection Gaps Identified

### Gap 1 — Unrecoverable Silence Window
Actions taken while CloudTrail is stopped leave no recoverable evidence
in CloudTrail logs. The gap is detectable but its contents are not.

### Gap 2 — GuardDuty Detection Lag
GuardDuty finding generation has a 5-15 minute delay after the triggering
event. A fast attacker can complete their objective before any alert fires.

### Gap 3 — Finding Aggregation Hides Frequency
GuardDuty incremented the count on the existing finding rather than
creating a new one. A SOC analyst reviewing findings sees count: 3
rather than a new high-priority alert — reducing urgency perception.

## Recommended Detections

| Detection | Implementation | Covers |
|-----------|---------------|--------|
| EventBridge rule on StopLogging | Auto-escalate to Critical | Gap 2 |
| CloudTrail integrity validation | AWS CloudTrail log validation | Gap 1 |
| GuardDuty finding EventBridge | Real-time SNS on new findings | Gap 2 |
| AWS Config rule for CloudTrail | Alert on trail status change | Gap 1 |

## Kill Chain Summary

This scenario demonstrates a complete attacker sequence:

1. Initial Access — valid credentials obtained
2. Discovery — ScoutSuite reconnaissance (Scenario recon phase)
3. Defense Evasion — CloudTrail stopped (T1562.008)
4. Discovery — IAM and S3 enumeration under silence (T1087.004, T1580)
5. Defense Evasion — CloudTrail restarted to hide gap

## Azure Equivalent

| AWS | Azure |
|-----|-------|
| CloudTrail StopLogging | Disable Azure Monitor Diagnostic Settings |
| Silence window enumeration | Azure Resource Graph queries |
| CloudTrail gap detection | Azure Monitor Activity Log gaps |
| EventBridge escalation | Azure Monitor Alert Rules |

## Status
Scenario 3 Complete — Reconnaissance under silence demonstrated,
silence gap verified in CloudTrail, detection gaps documented.