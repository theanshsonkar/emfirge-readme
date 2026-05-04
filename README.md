# Emfirge

AI cloud security agent. Scans your AWS infrastructure, maps attack paths, simulates breaches, and auto-remediates with one-click Terraform PRs.

**Free during beta.** → [emfirge.cloud](https://emfirge.cloud)

---

## How it works

1. **Deploy a read-only IAM role** into your AWS account (60 seconds)
2. **Paste the Role ARN** into Emfirge
3. **Get a full security report** — findings, attack paths, toxic combos, risk scores, and AI-powered remediation

Emfirge assumes the role, scans 16 AWS services, and returns results in under 30 seconds. No credentials are stored. No write access. You can revoke access instantly by deleting the CloudFormation stack.

---

## Quick start

### One-click deploy

Click below to deploy the IAM role directly from the AWS Console:

[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/create/review?templateURL=https://emfirge-reports.s3.ap-south-1.amazonaws.com/cloudformation/iam-role.yaml&stackName=EmfirgeReadOnlyStack)

Or go to [emfirge.cloud/scan.html](https://emfirge.cloud/scan.html) — the connect flow walks you through it step by step.

### CLI deploy

```bash
aws cloudformation deploy \
  --template-file iam-role.yaml \
  --stack-name emfirge-access \
  --capabilities CAPABILITY_NAMED_IAM
```

Get the Role ARN:

```bash
aws cloudformation describe-stacks \
  --stack-name emfirge-access \
  --query "Stacks[0].Outputs[?OutputKey=='RoleArn'].OutputValue" \
  --output text
```

Then paste it at [emfirge.cloud](https://emfirge.cloud).

---

## What Emfirge scans

58 security rules across 16 AWS services:

**Compute** — EC2, Lambda, ECS
**Storage** — S3, EBS
**Database** — RDS
**Identity** — IAM, Secrets Manager, KMS
**Network** — VPC, Security Groups, WAF, CloudFront, SNS
**Observability** — CloudTrail, GuardDuty, CloudWatch, AWS Config
**Cost** — Budgets, billing alarms, orphaned resources

Each finding includes severity, confidence, blast radius, attack path, and MITRE ATT&CK mapping.

---

## What the IAM role grants

A single read-only role (`EmfirgeReadOnlyRole`) with:

| Permission | Why |
|---|---|
| `SecurityAudit` (managed policy) | Read EC2, S3, RDS, IAM, VPC, and 30+ services |
| GuardDuty | Check if threat detection is enabled |
| Secrets Manager | List secrets (no read access to values) |
| Lambda | Scan function configs and attached roles |
| ECS | Detect privileged containers and task roles |
| KMS | Check key rotation status |
| AWS Config | Check compliance recording |
| WAF | Check if web ACLs protect your ALBs |
| CloudFront | Detect distributions for S3 context-aware rules |
| Budgets | Detect if cost alerts are configured |
| SNS | Check topic encryption and public access |

**No write permissions. No data access. No credential storage.**

---

## Security model

- **Read-only** — cannot create, modify, or delete any resource
- **External ID** — prevents confused deputy attacks
- **Scoped trust** — only Emfirge's AWS account (`282027772803`) can assume the role
- **Temporary credentials** — STS tokens expire after the scan, never stored
- **Instant revoke** — delete the CloudFormation stack and all access is gone

Full details: [emfirge.cloud/security.html](https://emfirge.cloud/security.html)

---

## Revoke access

```bash
aws cloudformation delete-stack --stack-name emfirge-access
```

Or in the AWS Console: CloudFormation → select the stack → Delete.

The role is permanently removed. Emfirge loses access immediately.

---

## Links

- **Product** — [emfirge.cloud](https://emfirge.cloud)
- **Trust & Security** — [emfirge.cloud/security.html](https://emfirge.cloud/security.html)
- **About** — [emfirge.cloud/about.html](https://emfirge.cloud/about.html)

Built by [Ansh Sonkar](https://github.com/theanshsonkar) · Founders, Inc. Canopy 2026
