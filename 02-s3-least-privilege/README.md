# Lab: Creating a Policy to Allow the IAM User to Read an S3 Object

## Objective

Create an inline IAM policy that allows the `iam-lab-user` to read objects inside the `Sensitive-data/` path of the S3 bucket.

---

## Step 1 — Create the Policy

Go to:

**AWS Console → IAM → Users → `iam-lab-user` → Add permissions → Create inline policy**

Give the policy a meaningful name:

```text
S3ReadLabObjects
```

---

## Step 2 — Define the Policy

Open the **JSON editor** and add the following policy:

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

### What the policy means

```text
Effect
└── Allow

Action
└── s3:GetObject

Resource
└── gad-iam-security-lab-2026/Sensitive-data/*
```

The policy allows the IAM user to **read objects** under the `Sensitive-data/` prefix.

It does **not** allow the user to:

* Upload objects
* Delete objects
* Modify objects
* Access objects outside the specified path

---

## Step 3 — Test Object Read Access

Using the AWS CLI profile for the lab user:

```powershell
aws s3 cp s3://gad-iam-security-lab-2026/Sensitive-data/employee_data.txt . --profile iam-lab
```

The download should succeed.

Verify the downloaded file:

```powershell
Get-Content .\employee_data.txt
```

---

## Security Concept

This demonstrates the difference between **bucket-level** and **object-level** permissions.

The first policy allowed:

```text
s3:ListBucket
```

against the bucket:

```text
arn:aws:s3:::gad-iam-security-lab-2026
```

The second policy allows:

```text
s3:GetObject
```

against objects:

```text
arn:aws:s3:::gad-iam-security-lab-2026/Sensitive-data/*
```

This demonstrates how AWS IAM can restrict permissions to the specific resources and actions that a user actually needs.
