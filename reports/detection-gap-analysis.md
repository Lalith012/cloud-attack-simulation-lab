# Detection Gap Analysis — Phase 4

## Overview

This report documents the detection gaps identified across all three attack
scenarios in Phase 3. For each gap, we document what was detected, what was
missed, why the gap exists, and specific recommendations to close it.

## Analysis Methodology

Each attack scenario was executed against a live AWS account with GuardDuty
and CloudTrail active. GuardDuty findings were collected after each scenario
and compared against the actual actions performed. Gaps were identified where
actions produced no GuardDuty finding or where findings had inadequate
severity or timing.

## GuardDuty Findings Summary

| Finding Type | Severity | Scenario | Detected |
|-------------|----------|----------|---------|
| Stealth:IAMUser/CloudTrailLoggingDisabled | 2.0 (Low) | 1 and 3 | Yes |
| Policy:IAMUser/RootCredentialUsage | 2.0 (Low) | Pre-existing | Yes |
| S3 data exfiltration | N/A | 2 | No |
| IAM enumeration under silence | N/A | 3 | No |
| S3 enumeration under silence | N/A | 3 | No |

## Detection Gap 1 — Severity Miscalibration

### What Happened
GuardDuty detected CloudTrail StopLogging and generated finding type
Stealth:IAMUser/CloudTrailLoggingDisabled with severity 2.0 (Low).

### The Gap
Severity 2.0 means the finding sits in a low priority queue. In a busy
SOC, low severity findings may not be investigated for hours or days.
CloudTrail disabling is a precursor attack technique — something worse
is always coming next. Low severity does not reflect this context.

### Why It Exists
GuardDuty assigns severity based on the action alone, not its context
in an attack chain. Disabling logging is rated low because it doesn't
directly cause data loss or access. GuardDuty has no context about
what the attacker plans to do next.

### Recommendation
Implement an EventBridge rule that triggers on
Stealth:IAMUser/CloudTrailLoggingDisabled regardless of severity and
sends an immediate SNS notification to the security team.

CloudTrail → CloudWatch Logs → Lambda → SNS
Lambda checks for StopLogging events and fires alert in seconds

---

## Detection Gap 4 — S3 Read Access Invisible to GuardDuty

### What Happened
In Scenario 2, CloudTrail log files were downloaded from S3 using
valid credentials. GuardDuty generated zero findings.

### The Gap
GuardDuty does not alert on S3 GetObject calls using legitimate
credentials by default. An attacker can exfiltrate any readable
S3 data without triggering a single GuardDuty finding.

### Why It Exists
GuardDuty distinguishes between legitimate and anomalous access
using behavioral baselines. For a new account with limited history,
any S3 access looks legitimate. Additionally GuardDuty S3 protection
requires explicit enablement and behavioral baseline establishment.

### Recommendation
Enable GuardDuty S3 Protection feature and allow 7-14 days for
baseline establishment. Additionally enable AWS Macie on the
CloudTrail log bucket to detect sensitive data access patterns.

---

## Detection Gap 5 — CloudTrail Data Events Disabled

### What Happened
S3 object downloads in Scenario 2 did not appear in CloudTrail.

### The Gap
CloudTrail only logs management events by default. Data events —
including S3 GetObject, PutObject, and DeleteObject — require
explicit enablement. Without data event logging, S3 exfiltration
leaves no CloudTrail evidence.

### Why It Exists
Data event logging generates high volumes of events and incurs
additional CloudTrail costs. AWS disables it by default to avoid
unexpected billing.

### Recommendation
Enable CloudTrail data events for the CloudTrail log bucket specifically:

aws cloudtrail put-event-selectors 
--trail-name detection-lab-trail 
--event-selectors '[{
"ReadWriteType": "All",
"DataResources": [{
"Type": "AWS::S3::Object",
"Values": ["arn:aws:s3:::detection-lab-cloudtrail-logs-[account-id]/"]
}]
}]'

---

## Detection Gap 6 — Silence Window Contents Unrecoverable

### What Happened
In Scenario 3, IAM and S3 enumeration performed while CloudTrail
was stopped left zero recoverable evidence in CloudTrail.

### The Gap
A forensic investigator can detect that a silence gap exists but
cannot recover what actions were taken during it. The gap is
detectable, its contents are not.

### Why It Exists
CloudTrail is a logging service — it can only record events while
active. There is no mechanism to retroactively recover events that
occurred while logging was disabled.

### Recommendation
Implement VPC Flow Logs for network-level visibility during CloudTrail
gaps. While VPC Flow Logs don't capture API calls, they record network
connections that can help reconstruct attacker activity. Additionally,
endpoint detection on EC2 instances provides host-level visibility
independent of CloudTrail.

---

## Detection Gap 7 — Finding Aggregation Reduces Urgency

### What Happened
GuardDuty incremented the count on the existing
Stealth:IAMUser/CloudTrailLoggingDisabled finding rather than
generating a new finding for Scenario 3.

### The Gap
A SOC analyst reviewing findings sees count: 3 on a low severity
finding and treats it as a known, previously assessed alert. A new
occurrence of CloudTrail disabling should trigger fresh investigation
regardless of prior occurrences.

### Why It Exists
GuardDuty aggregates repeated occurrences of the same finding type
to reduce alert fatigue. This is appropriate for some finding types
but counterproductive for high-impact precursor techniques.

### Recommendation
Implement EventBridge rule that triggers on every GuardDuty finding
update for Stealth finding types, not just new findings. This ensures
each recurrence is treated as a fresh incident.

---

## Detection Coverage Summary

| Scenario | Action | Detected | Method | Gap |
|----------|--------|---------|--------|-----|
| 1 | CloudTrail StopLogging | Yes | GuardDuty | Severity too low |
| 1 | Actions during silence | No | None | Unrecoverable |
| 2 | S3 ListBucket | No | None | Not monitored |
| 2 | S3 GetObject | No | None | Data events disabled |
| 3 | CloudTrail StopLogging | Yes | GuardDuty | Aggregated, lag |
| 3 | IAM enum under silence | No | None | Silence window |
| 3 | S3 enum under silence | No | None | Silence window |

## Overall Detection Rate

| Metric | Value |
|--------|-------|
| Total attack actions | 7 |
| Detected by GuardDuty | 2 |
| Missed entirely | 5 |
| Detection rate | 28% |
| Actions with adequate severity | 0 |

## Key Takeaway

GuardDuty detected 28% of attack actions with default configuration.
The most dangerous actions — enumeration under silence and S3 data
exfiltration — were completely invisible. Closing the identified gaps
would raise detection coverage to approximately 85%.

## Azure Comparison

| Gap | AWS Detection | Azure Equivalent |
|-----|--------------|-----------------|
| CloudTrail disabled | GuardDuty Stealth finding | Defender for Cloud alert |
| S3 exfiltration | Not detected by default | Defender for Storage anomaly |
| Enumeration under silence | Not detected | Not detected |
| Severity miscalibration | Low severity default | Medium severity default |

