# Terraform Troubleshooting Guide

## Table of Contents

1. Terraform Command Not Found
2. Terraform Init Failed
3. Provider Installation Failed
4. Backend Initialization Error
5. State Lock Error
6. Terraform Validate Failed
7. Terraform Plan Failed
8. Terraform Apply Failed
9. Resource Already Exists
10. Invalid AWS Credentials
11. Access Denied Error
12. Terraform State Drift
13. State File Corruption
14. Resource Not Found
15. Dependency Cycle Error
16. Variable Not Defined
17. Unsupported Argument Error
18. Invalid Resource Type
19. Module Not Found
20. Remote State Access Failed
21. S3 Backend Issues
22. DynamoDB Lock Issues
23. Terraform Destroy Failed
24. Terraform Import Failed
25. Sensitive Data Exposure
26. Terraform Workspace Issues
27. Kubernetes Provider Issues
28. EKS Deployment Issues
29. ALB Creation Failed
30. EC2 Creation Failed
31. Scenario-Based Troubleshooting Questions
32. Useful Commands

---

# 1. Terraform Command Not Found

## Error

```bash
terraform: command not found
```

## Cause

Terraform is not installed.

## Solution

Verify installation:

```bash
terraform version
```

Install Terraform:

```bash
sudo apt update
sudo apt install unzip -y
```

Download Terraform:

```bash
wget https://releases.hashicorp.com/terraform/
```

---

# 2. Terraform Init Failed

## Error

```bash
terraform init
Error: Initialization failed
```

## Cause

- Backend issue
- Provider issue
- Internet connectivity

## Solution

```bash
terraform init -upgrade
```

Delete cache:

```bash
rm -rf .terraform
terraform init
```

---

# 3. Provider Installation Failed

## Error

```bash
Failed to install provider
```

## Cause

- Internet issue
- Wrong provider version

## Solution

Check connectivity:

```bash
ping registry.terraform.io
```

Update provider:

```hcl
required_providers {
 aws = {
   source = "hashicorp/aws"
   version = "~> 5.0"
 }
}
```

---

# 4. Backend Initialization Error

## Error

```bash
Backend initialization required
```

## Solution

```bash
terraform init -reconfigure
```

---

# 5. State Lock Error

## Error

```bash
Error acquiring the state lock
```

## Cause

Another Terraform process is running.

## Solution

Unlock state:

```bash
terraform force-unlock LOCK_ID
```

---

# 6. Terraform Validate Failed

## Error

```bash
terraform validate
```

Shows syntax errors.

## Solution

Format code:

```bash
terraform fmt
```

Validate:

```bash
terraform validate
```

---

# 7. Terraform Plan Failed

## Error

```bash
terraform plan
```

## Cause

- Missing variables
- Invalid configuration

## Solution

Check variables:

```bash
terraform plan -var-file=terraform.tfvars
```

---

# 8. Terraform Apply Failed

## Error

```bash
terraform apply
```

## Cause

Resource creation failure.

## Solution

Review logs:

```bash
TF_LOG=DEBUG terraform apply
```

---

# 9. Resource Already Exists

## Error

```bash
ResourceAlreadyExistsException
```

## Cause

Terraform tries to create existing resource.

## Solution

Import resource:

```bash
terraform import
```

Example:

```bash
terraform import aws_instance.web i-123456
```

---

# 10. Invalid AWS Credentials

## Error

```bash
InvalidClientTokenId
```

## Cause

Wrong AWS credentials.

## Solution

Verify:

```bash
aws sts get-caller-identity
```

Configure:

```bash
aws configure
```

---

# 11. Access Denied Error

## Error

```bash
AccessDenied
```

## Cause

Missing IAM permissions.

## Solution

Verify IAM role.

Example:

```json
AmazonEC2FullAccess
AmazonS3FullAccess
```

---

# 12. Terraform State Drift

## Cause

Resources modified manually.

## Solution

Detect drift:

```bash
terraform plan
```

Refresh state:

```bash
terraform refresh
```

---

# 13. State File Corruption

## Error

```bash
Failed to load state
```

## Solution

Restore backup:

```bash
terraform.tfstate.backup
```

---

# 14. Resource Not Found

## Error

```bash
ResourceNotFoundException
```

## Solution

Verify:

```bash
aws ec2 describe-instances
```

Check state:

```bash
terraform state list
```

---

# 15. Dependency Cycle Error

## Error

```bash
Cycle detected
```

## Cause

Circular dependency.

## Solution

Use:

```hcl
depends_on
```

Carefully.

---

# 16. Variable Not Defined

## Error

```bash
No value for required variable
```

## Solution

Define variable:

```hcl
variable "region" {}
```

Provide value:

```bash
terraform apply -var region=us-east-1
```

---

# 17. Unsupported Argument Error

## Error

```bash
Unsupported argument
```

## Cause

Wrong attribute.

## Solution

Check provider documentation.

---

# 18. Invalid Resource Type

## Error

```bash
Invalid resource type
```

## Example

Wrong:

```hcl
resource "aws_ec22" "web"
```

Correct:

```hcl
resource "aws_instance" "web"
```

---

# 19. Module Not Found

## Error

```bash
Module not installed
```

## Solution

```bash
terraform init
```

Verify source:

```hcl
source = "./modules/vpc"
```

---

# 20. Remote State Access Failed

## Error

```bash
Failed to load remote state
```

## Solution

Verify backend:

```hcl
backend "s3"
```

Check access permissions.

---

# 21. S3 Backend Issues

## Error

```bash
NoSuchBucket
```

## Solution

Verify bucket:

```bash
aws s3 ls
```

Create bucket:

```bash
aws s3 mb s3://terraform-state
```

---

# 22. DynamoDB Lock Issues

## Error

```bash
ConditionalCheckFailedException
```

## Cause

State lock stuck.

## Solution

Delete lock manually.

```bash
aws dynamodb scan --table-name terraform-lock
```

---

# 23. Terraform Destroy Failed

## Error

```bash
DependencyViolation
```

## Cause

Dependent resources exist.

## Solution

Destroy specific resource first.

```bash
terraform destroy -target=aws_instance.web
```

---

# 24. Terraform Import Failed

## Error

```bash
Cannot import non-existent resource
```

## Solution

Verify resource ID.

```bash
aws ec2 describe-instances
```

---

# 25. Sensitive Data Exposure

## Problem

Secrets stored in:

```hcl
password = "admin123"
```

## Solution

Use variables.

```hcl
sensitive = true
```

Use:

```bash
AWS Secrets Manager
HashiCorp Vault
```

---

# 26. Terraform Workspace Issues

## List Workspaces

```bash
terraform workspace list
```

## Switch Workspace

```bash
terraform workspace select prod
```

## Create Workspace

```bash
terraform workspace new dev
```

---

# 27. Kubernetes Provider Issues

## Error

```bash
Kubernetes cluster unreachable
```

## Cause

Invalid kubeconfig.

## Solution

Verify:

```bash
kubectl get nodes
```

Update config:

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name my-cluster
```

---

# 28. EKS Deployment Issues

## Error

```bash
Node group failed
```

## Check

```bash
aws eks describe-nodegroup
```

Verify:

- IAM Role
- Subnets
- Security Groups

---

# 29. ALB Creation Failed

## Error

```bash
failed to reconcile
```

## Cause

AWS Load Balancer Controller missing.

## Verify

```bash
kubectl get pods -n kube-system
```

Check IAM role.

---

# 30. EC2 Creation Failed

## Error

```bash
InsufficientInstanceCapacity
```

## Solution

Change:

```hcl
instance_type = "t3.medium"
```

Or use another AZ.

---

# 31. Scenario-Based Troubleshooting Questions

## Scenario 1

Terraform apply fails with:

```bash
InvalidClientTokenId
```

### What will you check?

- AWS credentials
- IAM permissions
- AWS CLI access

---

## Scenario 2

Terraform state locked.

### What will you do?

```bash
terraform force-unlock LOCK_ID
```

Verify no active runs.

---

## Scenario 3

Developer manually deleted EC2.

### What happens?

Terraform state drift occurs.

Fix:

```bash
terraform plan
terraform apply
```

---

## Scenario 4

Terraform cannot connect to EKS.

### Check

```bash
kubectl get nodes
```

Verify kubeconfig.

---

## Scenario 5

terraform init suddenly fails.

### Check

- Backend
- Internet
- Provider registry

---

## Scenario 6

S3 backend inaccessible.

### Check

- Bucket exists
- IAM permissions
- Region mismatch

---

## Scenario 7

Terraform apply hangs forever.

### Check

```bash
TF_LOG=DEBUG terraform apply
```

Review resource creation logs.

---

## Scenario 8

ALB not created after apply.

### Check

- Ingress annotations
- AWS Load Balancer Controller
- IAM permissions

---

## Scenario 9

Resource exists but Terraform wants to recreate it.

### Solution

Import resource.

```bash
terraform import
```

---

## Scenario 10

Production state file accidentally deleted.

### Recovery

Restore:

```bash
terraform.tfstate.backup
```

Or recover from S3 backend.

---

## Scenario 11

Developer changed resource manually in AWS Console.

### Result

State drift.

### Fix

```bash
terraform refresh
terraform plan
```

---

## Scenario 12

Multiple engineers running Terraform simultaneously.

### Solution

Use:

- S3 Backend
- DynamoDB State Locking

---

## Scenario 13

terraform destroy fails.

### Check

Dependencies.

Use:

```bash
terraform destroy -target
```

---

## Scenario 14

Module code updated but not reflected.

### Solution

```bash
terraform init -upgrade
```

---

## Scenario 15

Terraform succeeds but application not accessible.

### Check

- Security Groups
- Route Tables
- NACL
- Load Balancer
- DNS

---

# 32. Useful Commands

## Initialize

```bash
terraform init
```

## Validate

```bash
terraform validate
```

## Format

```bash
terraform fmt
```

## Plan

```bash
terraform plan
```

## Apply

```bash
terraform apply
```

## Destroy

```bash
terraform destroy
```

## Show State

```bash
terraform show
```

## State List

```bash
terraform state list
```

## Import Resource

```bash
terraform import
```

## Refresh State

```bash
terraform refresh
```

## Force Unlock

```bash
terraform force-unlock LOCK_ID
```

## Workspace List

```bash
terraform workspace list
```

## Workspace Switch

```bash
terraform workspace select prod
```

---

# Terraform Troubleshooting Workflow

```text
Terraform Error
        |
        v
Read Error Message
        |
        v
terraform validate
        |
        v
terraform plan
        |
        v
Check State File
        |
        v
Check Backend
        |
        v
Check IAM Permissions
        |
        v
Check Cloud Resources
        |
        v
Fix Issue
        |
        v
terraform apply
```
