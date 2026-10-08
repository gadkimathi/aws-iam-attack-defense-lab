# AWS IAM Attack & Defense Lab

A hands-on AWS security laboratory for learning and demonstrating **Identity and Access Management (IAM), least privilege, S3 access controls, IAM roles, privilege escalation, detection, and remediation**.

The lab is built progressively to simulate common cloud security scenarios in a controlled AWS environment using synthetic data.

---

## Objectives

* Understand AWS IAM users, roles, policies, and permissions
* Apply the principle of least privilege
* Understand S3 bucket-level vs object-level permissions
* Analyze `AccessDenied` errors
* Explore IAM role trust policies and `AssumeRole`
* Simulate IAM privilege-escalation paths
* Detect suspicious IAM activity using CloudTrail
* Use IAM Access Analyzer to identify excessive permissions
* Implement security remediation
* Reproduce the environment using Terraform

---

## Architecture

```text
                    AWS Account
                         │
                         │
                  ┌──────▼──────┐
                  │ IAM User    │
                  │ iam-lab-user│
                  └──────┬──────┘
                         │
                   AWS CLI Profile
                     iam-lab
                         │
                         ▼
              ┌─────────────────────┐
              │     Amazon S3       │
              │                     │
              │ gad-iam-security-  │
              │ lab-2026            │
              │                     │
              │ └─ Sensitive-data/ │
              │    └─ employee_     │
              │       data.txt      │
              └─────────────────────┘
```

---

# Phase 1 — IAM User & Least Privilege

## 1. IAM User

Created a dedicated IAM user for the laboratory:

```text
iam-lab-user
```

The user was configured for **programmatic access only** and used through the AWS CLI.

The lab does not use the root account for normal testing.

---

## 2. AWS CLI Profile

A dedicated AWS CLI profile was configured:

```bash
aws configure --profile iam-lab
```

Identity verification:

```bash
aws sts get-caller-identity --profile iam-lab
```

This confirms that commands are being executed using the laboratory IAM identity rather than another AWS identity.

---

# Phase 2 — S3 Least Privilege

## S3 Bucket

Created a private S3 bucket:

```text
gad-iam-security-lab-2026
```

The bucket contains synthetic laboratory data:

```text
Sensitive-data/
└── employee_data.txt
```

No real employee or confidential information is used.

---

## Initial Permission

The IAM user was initially granted only:

```text
s3:ListBucket
```

against the bucket:

```text
arn:aws:s3:::gad-iam-security-lab-2026
```

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::gad-iam-security-lab-2026"
    }
  ]
}
```

### Test

The user could list the bucket:

```bash
aws s3 ls s3://gad-iam-security-lab-2026 --profile iam-lab
```

Result:

```text
PRE Sensitive-data/
```

However, attempting to download an object resulted in:

```text
403 Forbidden
```

This demonstrated that:

> `s3:ListBucket` does not automatically grant permission to read objects.

---

# Phase 3 — Object-Level Access

A second policy was created to allow:

```text
s3:GetObject
```

only for objects under:

```text
Sensitive-data/*
```

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::gad-iam-security-lab-2026/Sensitive-data/*"
    }
  ]
}
```

The user could then successfully download:

```bash
aws s3 cp s3://gad-iam-security-lab-2026/Sensitive-data/employee_data.txt . --profile iam-lab
```

---

# Least-Privilege Testing

The permissions were deliberately tested rather than assumed.

| S3 Action         | Result    |
| ----------------- | --------- |
| `s3:ListBucket`   | ✅ Allowed |
| `s3:GetObject`    | ✅ Allowed |
| `s3:PutObject`    | ❌ Denied  |
| `s3:DeleteObject` | ❌ Denied  |

### PutObject test

```bash
aws s3 cp ./iam-write-test.txt s3://gad-iam-security-lab-2026/Sensitive-data/iam-write-test.txt --profile iam-lab
```

Result:

```text
AccessDenied
```

AWS reported:

```text
no identity-based policy allows the s3:PutObject action
```

### DeleteObject test

```bash
aws s3 rm s3://gad-iam-security-lab-2026/Sensitive-data/iam-write-test.txt --profile iam-lab
```

Result:

```text
AccessDenied
```

AWS reported:

```text
no identity-based policy allows the s3:DeleteObject action
```

---

# Key Security Finding

This phase demonstrates the difference between **bucket-level and object-level permissions**.

```text
s3:ListBucket
       │
       ▼
Can discover objects

s3:GetObject
       │
       ▼
Can read objects

s3:PutObject
       │
       ▼
Can upload/modify objects

s3:DeleteObject
       │
       ▼
Can delete objects
```

Granting one permission does not automatically grant the others.

This is an example of implementing the **principle of least privilege**.

---

# Security Concepts Demonstrated

* AWS IAM users
* Identity-based policies
* IAM policy statements
* Allow permissions
* Explicit resource scoping
* S3 bucket ARNs
* S3 object ARNs
* Least privilege
* AccessDenied troubleshooting
* AWS CLI authentication
* S3 access control

---

# Lab Roadmap

The laboratory will progressively expand into more advanced AWS security scenarios.

### Phase 1 — IAM User & Least Privilege

* [x] Create IAM user
* [x] Configure AWS CLI
* [x] Create S3 bucket
* [x] Apply `ListBucket`
* [x] Apply `GetObject`
* [x] Test denied `PutObject`
* [x] Test denied `DeleteObject`

### Phase 2 — IAM Roles

* [ ] Create IAM role
* [ ] Understand trust policies
* [ ] Understand permission policies
* [ ] Assume role using STS
* [ ] Work with temporary credentials

### Phase 3 — IAM Privilege Escalation

* [ ] Identify excessive permissions
* [ ] Explore `iam:PassRole`
* [ ] Analyze privilege-escalation paths
* [ ] Demonstrate controlled escalation in the lab
* [ ] Remediate excessive permissions

### Phase 4 — Permission Boundaries

* [ ] Create permission boundary
* [ ] Test maximum permission limits
* [ ] Compare identity policies vs permission boundaries

### Phase 5 — Detection

* [ ] Enable CloudTrail
* [ ] Generate IAM activity
* [ ] Analyze CloudTrail events
* [ ] Identify suspicious IAM actions

### Phase 6 — IAM Access Analyzer

* [ ] Analyze external access
* [ ] Identify unintended access
* [ ] Review generated findings
* [ ] Remediate excessive access

### Phase 7 — Infrastructure as Code

* [ ] Rebuild the lab with Terraform
* [ ] Define IAM policies as code
* [ ] Define S3 infrastructure as code
* [ ] Apply security controls through Terraform

---

# Tools

* AWS IAM
* Amazon S3
* AWS STS
* AWS CloudTrail
* IAM Access Analyzer
* AWS CLI
* PowerShell
* Terraform
* Git & GitHub

---

# Security Disclaimer

This repository contains a controlled security laboratory designed for learning AWS IAM security concepts.

All attack simulations are intended to be performed only against AWS resources owned or explicitly authorized by the lab operator.

The laboratory uses synthetic data and does not intentionally target third-party systems.

---

## Author

**Gad Kimathi**

Mechatronics Engineer | Cloud Security | IAM | AWS | Kubernetes

GitHub: `gadkimathi`
