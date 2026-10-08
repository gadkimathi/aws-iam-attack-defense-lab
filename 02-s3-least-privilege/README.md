## Lab for creating a policy for allowing the IAM user to read the Object

# First step
Create the policy. Create the inline policy and name it.
# Second step
Open the json editor and write the json to allow the IAM user to read the object from the s3 bucket.

```json
{
"Version":"2012-10-17",
"Statement":[
{
"Effect":"Allow",
"Action":"s3:GetObject",
"Resource":"arn:aws:s3:::gad-iam-security-lab-2026/Sensitive-data/*"
}
]
}
