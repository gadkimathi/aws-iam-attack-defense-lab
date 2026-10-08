# Phase 1 — IAM User

## Objective

Create a dedicated AWS IAM user for the security laboratory and configure controlled programmatic access through the AWS CLI.

## IAM User

```text
iam-lab-user
```

The user was created specifically for this laboratory and does not have AWS Management Console access.

## AWS CLI Configuration

A dedicated AWS CLI profile was configured:

```powershell
aws configure --profile iam-lab
```

The active identity was verified with:

```powershell
aws sts get-caller-identity --profile iam-lab
```

This confirmed that commands executed using the `iam-lab` profile run under the dedicated IAM user.

## Initial Permission

The user was intentionally given only the following permission:

```text
s3:ListBucket
```

The permission is restricted to the laboratory S3 bucket.

Policy:

```json
{
"Version":"2012-10-17",
"Statement":[
{
"Effect":"Allow",
"Action":"s3:ListBucket",
"Resource":"arn:aws:s3:::gad-iam-security-lab-2026"
}
]
}
```
