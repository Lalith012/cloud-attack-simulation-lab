# Phase 3 — Attack Simulation: Kill Chain

## Overview

This phase covers the execution of three attack scenarios against the target
AWS account. Each scenario was designed based on attack paths identified
during Phase 2 reconnaissance. All attacks were conducted on infrastructure
owned by the author in an authorized lab environment.

## Attack Scenarios Executed

### Scenario 1 — Defense Evasion: CloudTrail Disruption
- Technique: T1562.008 — Impair Defenses: Disable Cloud Logs
- Action: cloudtrail:StopLogging executed against detection-lab-trail
- Result: IsLogging set to false — audit trail eliminated
- GuardDuty Response: Stealth:IAMUser/CloudTrailLoggingDisabled generated
- Severity: 2.0 (Low) — detection gap identified
- Full documentation: attacks/04-scenario1-cloudtrail-disruption.md

### Scenario 2 — Data Exfiltration: CloudTrail Log Theft
- Technique: T1530 — Data from Cloud Storage
- Action: CloudTrail log file downloaded and parsed from S3
- Result: Account intelligence extracted — identity, active hours, environment
- GuardDuty Response: Zero findings generated
- Detection gap: S3 GetObject invisible to GuardDuty by default
- Full documentation: attacks/05-scenario2-data-exfiltration.md

### Scenario 3 — Reconnaissance Under Silence
- Techniques: T1562.008, T1087.004, T1580
- Action: CloudTrail stopped, IAM and S3 enumerated, CloudTrail restarted
- Result: Silence gap verified — enumeration left zero CloudTrail evidence
- GuardDuty Response: Existing finding count incremented — no new finding
- Detection gap: Silence window contents unrecoverable
- Full documentation: attacks/06-scenario3-recon-under-silence.md

## Kill Chain Summary

| Stage | Action | Technique |
|-------|--------|-----------|
| Reconnaissance | ScoutSuite account audit | T1526, T1580 |
| Reconnaissance | Pacu IAM enumeration | T1087.004 |
| Reconnaissance | S3 bucket enumeration | T1619 |
| Defense Evasion | CloudTrail StopLogging | T1562.008 |
| Collection | CloudTrail log download | T1530 |
| Discovery | IAM enum under silence | T1087.004 |
| Discovery | S3 enum under silence | T1619 |

## GuardDuty Findings Generated

| Finding Type | Severity | Generated |
|-------------|----------|-----------|
| Stealth:IAMUser/CloudTrailLoggingDisabled | 2.0 Low | Yes |
| S3 exfiltration finding | N/A | No |
| Enumeration under silence | N/A | No |

## Overall Detection Rate
- Total attack actions: 7
- Detected: 2 (28%)
- Missed: 5 (72%)

## Safety Controls Applied
- CloudTrail logging restored immediately after each StopLogging demonstration
- No data permanently deleted or modified
- All destructive actions discussed and confirmed before execution
- Environment verified clean after each scenario

## MITRE ATT&CK Coverage

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1562.008 | Impair Defenses: Disable Cloud Logs | Defense Evasion |
| T1530 | Data from Cloud Storage | Collection |
| T1087.004 | Account Discovery: Cloud Account | Discovery |
| T1580 | Cloud Infrast