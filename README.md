# Emfirge — AWS Access Setup

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

This repository contains the CloudFormation template that creates a **read-only IAM role** in your AWS account. Emfirge assumes this role to scan your infrastructure and generate a security risk report.

**No write permissions are granted. You can delete the stack at any time to revoke all access instantly.**

---

## What it creates

A single IAM role (`EmfirgeReadOnlyRole`) with:

- **AWS SecurityAudit** managed policy — standard read-only access across AWS services
- Additional read permissions for: GuardDuty, Secrets Manager, Lambda, ECS, WAF, KMS, AWS Config, CloudFront, SNS, Budgets
- A trust policy that allows only Emfirge's AWS account to assume this role
- An **External ID** condition — an extra security layer that prevents confused deputy attacks

All permissions are read-only. Emfirge cannot create, modify, or delete any resource in your account.

---

## Deploy

### Option 1 — AWS Console (recommended)

1. Open the [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation)
2. Click **Create stack → With new resources**
3. Select **Upload a template file** → upload `iam-role.yaml`
4. Click through the steps — no parameters need to be changed
5. Check the IAM acknowledgement box and click **Create stack**
6. Once the stack shows `CREATE_COMPLETE`, open the **Outputs** tab
7. Copy the **RoleArn** value

### Option 2 — AWS CLI

```bash
aws cloudformation deploy \
  --template-file iam-role.yaml \
  --stack-name emfirge-access \
  --capabilities CAPABILITY_NAMED_IAM
```

Then get the ARN:

```bash
aws cloudformation describe-stacks \
  --stack-name emfirge-access \
  --query "Stacks[0].Outputs[?OutputKey=='RoleArn'].OutputValue" \
  --output text
```

---

## Connect to Emfirge

1. Go to [emfirge.vercel.app](https://emfirge.vercel.app)
2. Paste the **Role ARN** from the CloudFormation output
3. Click **Start Scan**

Emfirge will assume the role, scan your infrastructure, and return a full security risk report.

---

## Permissions explained

| Permission group | Why Emfirge needs it |
|---|---|
| SecurityAudit (managed) | Read EC2, S3, RDS, IAM, VPC, and 30+ services |
| GuardDuty | Check if threat detection is enabled |
| Secrets Manager | Detect secrets without read access (list only) |
| Lambda | Scan function configs and attached roles |
| ECS | Detect task roles and container permissions |
| KMS | Check key rotation status |
| AWS Config | Check if compliance recording is active |
| WAF | Check if web ACLs are attached to resources |
| CloudFront | Detect distributions without WAF or HTTPS |
| Budgets | Detect if cost alerts are configured |
| SNS | Check alert notification setup |

---

## Revoke access

Delete the CloudFormation stack at any time:

```bash
aws cloudformation delete-stack --stack-name emfirge-access
```

Or in the AWS Console: CloudFormation → select `emfirge-access` → Delete.

The role is permanently removed. Emfirge loses access immediately.

---

## Security

- **Read-only** — no permissions to create, modify, or delete resources
- **External ID** — prevents third-party impersonation (confused deputy attack)
- **Scoped trust** — only Emfirge's AWS account can assume this role
- **No credentials stored** — Emfirge uses temporary STS tokens, never long-lived keys
- **Deletable** — one command removes all access permanently
