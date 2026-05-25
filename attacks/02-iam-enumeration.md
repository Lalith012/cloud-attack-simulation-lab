# IAM Enumeration — Pacu and AWS CLI Analysis

## Overview

Following ScoutSuite reconnaissance, Pacu and AWS CLI were used to perform
deep IAM enumeration. The goal was to identify privilege escalation paths,
assumable roles, and dangerous permission combinations in the target account.

## MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|-------------|------|--------|
| T1087.004 | Account Discovery: Cloud Account | Discovery |
| T1069.003 | Permission Groups Discovery: Cloud Groups | Discovery |
| T1078.004 | Valid Accounts: Cloud Accounts | Defense Evasion, Persistence |

## Tools Used

| Tool | Module/Command | Purpose |
|------|---------------|---------|
| Pacu | iam__enum_users_roles_policies_groups | Enumerate all IAM entities |
| Pacu | iam__enum_roles | Attempt role enumeration (blocked) |
| AWS CLI | iam list-roles | Retrieve role details |
| AWS CLI | iam list-attached-user-policies | Identify attached policies |
| AWS Console | IAM Policy JSON | Read policy permissions |

## IAM Entity Summary

| Entity Type | Count | Notes |
|-------------|-------|-------|
| Users | 1 | Lalith only — single user account |
| Roles | 6 | All service-linked roles |
| Policies | 1 | DetectionLabPolicy — custom least privilege |
| Groups | 0 | No group structure |

## Role Analysis

All 6 roles identified are AWS Service-Linked Roles — automatically created
by AWS when services are enabled. These roles cannot be assumed by human users
or external credentials.

| Role | Service | Assumable |
|------|---------|-----------|
| AWSServiceRoleForAmazonGuardDuty | GuardDuty | No |
| AWSServiceRoleForAmazonGuardDutyMalwareProtection | GuardDuty Malware | No |
| AWSServiceRoleForResourceExplorer | Resource Explorer | No |
| AWSServiceRoleForSecurityHub | Security Hub | No |
| AWSServiceRoleForSupport | AWS Support | No |
| AWSServiceRoleForTrustedAdvisor | Trusted Advisor | No |

**Privilege escalation via role assumption: No viable path identified.**

## Policy Permission Analysis

DetectionLabPolicy grants the following permissions across 6 service categories.
Analysis focuses on attacker-relevant permissions only.

### Critical Permissions Identified

| Permission | Service | Risk Level | Attack Use |
|------------|---------|------------|-----------|
| cloudtrail:StopLogging | CloudTrail | CRITICAL | Disable audit logging before attack |
| cloudtrail:DeleteTrail | CloudTrail | CRITICAL | Permanently remove trail |
| s3:PutBucketPolicy | S3 | HIGH | Modify bucket access controls |
| s3:PutBucketPublicAccessBlock | S3 | HIGH | Remove public access restrictions |
| s3:DeleteObject | S3 | HIGH | Destroy evidence or data |
| s3:DeleteBucket | S3 | HIGH | Destroy entire bucket |
| s3:PutBucketAcl | S3 | MEDIUM | Change bucket ownership settings |

### Privilege Escalation via IAM Policy Manipulation

The following dangerous IAM permissions are NOT present in DetectionLabPolicy:

| Missing Permission | Why It Matters |
|-------------------|----------------|
| iam:AttachUserPolicy | Cannot attach admin policy to self |
| iam:CreatePolicyVersion | Cannot create elevated policy version |
| iam:PutUserPolicy | Cannot add inline policy to self |
| iam:PassRole | Cannot pass roles to services |

**Result: No IAM-based privilege escalation path available.**
Least privilege policy effectively blocks this attack vector.

## Most Dangerous Permission Combination

cloudtrail:StopLogging + s3:DeleteObject + s3:PutBucketPolicy

Attack chain this enables:
1. Stop CloudTrail logging — eliminates API audit trail
2. GuardDuty partially blinded — loses CloudTrail as data source
3. Modify S3 bucket policies — enables data exfiltration or exposure
4. Delete S3 objects — destroy evidence of attack

This combination forms the basis of Phase 3 Attack Scenario 1.

## Detection Gap Identified

IAM enumeration was performed entirely using legitimate AWS API calls.
GuardDuty did not generate findings for any enumeration activity.

Read-only enumeration using valid credentials is invisible to GuardDuty
by default. This is documented as Detection Gap 1 in Phase 4.

## Attacker Conclusion

| Attack Vector | Viable | Reason |
|--------------|--------|--------|
| Role assumption | No | All roles are service-linked |
| IAM policy manipulation | No | Missing required IAM permissions |
| CloudTrail disruption | Yes | StopLogging and DeleteTrail permitted |
| S3 data manipulation | Yes | Multiple S3 write/delete permissions |

Primary attack paths identified: CloudTrail disruption followed by S3 manipulation.
Proceeding to Phase 3 attack simulation.