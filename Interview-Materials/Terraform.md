# 🏗️ Terraform — Complete Interview Guide

> **Language:** `HCL` · `Shell (Bash)` · `YAML` 

---

## 📁 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [What is Terraform](#1-what-is-terraform) | IaC, HCL, providers, workflow |
| 2 | [Core Commands](#2-core-commands) | init, plan, apply, destroy |
| 3 | [State Management](#3-state-management) | Local vs remote, locking, import |
| 4 | [Variables & Outputs](#4-variables--outputs) | Types, precedence, sensitive |
| 5 | [Modules](#5-modules) | Root vs child, sources, providers |
| 6 | [Expressions & Functions](#6-expressions--functions) | locals, count, for_each, functions |
| 7 | [Workspaces & Environments](#7-workspaces--environments) | Strategies, workspaces |
| 8 | [Advanced Concepts](#8-advanced-concepts) | lifecycle, dynamic, moved, check |
| 9 | [Security](#9-security) | Secrets, sensitive, encryption |
| 10 | [CI/CD Integration](#10-cicd-integration) | Pipelines, testing, cost control |
| 11 | [Scenario-Based Questions](#11-scenario-based-questions) | 55 real-world scenarios |
| 12 | [Quick Reference](#12-quick-reference) | Commands cheatsheet |

---


## 1. What is Terraform

### Q1. What is Terraform and why is it used?

Terraform is an open-source **Infrastructure as Code (IaC)** tool by HashiCorp. You write code to define, provision, and manage infrastructure across multiple cloud providers.

```
Without Terraform:  Manual clicks in AWS Console → inconsistent, error-prone
With Terraform:     Code → Plan → Apply → Repeatable infrastructure
```

**Why used:**

| Benefit | Description |
|---------|-------------|
| **Multi-cloud** | AWS, Azure, GCP, K8s, GitHub, 1000+ providers |
| **Declarative** | Describe desired state, not steps |
| **Idempotent** | Same config → same result, every time |
| **Version control** | Infrastructure tracked in Git |
| **Plan before apply** | Preview changes (dry run) |
| **Modular** | Reusable modules for common patterns |

**Terraform Workflow:**
```
┌─────────────────────────────────────────────────────────┐
│                  TERRAFORM WORKFLOW                     │
│                                                         │
│  Write Code  →  terraform init  →  terraform validate  │
│       │                                    │            │
│       └──── terraform plan  ────────────►  Review      │
│                    │                                    │
│             terraform apply  →  Infrastructure         │
│                    │                                    │
│             terraform.tfstate  →  State tracking       │
└─────────────────────────────────────────────────────────┘
```

---

### Q2. What is Infrastructure as Code (IaC)?

IaC is managing infrastructure through machine-readable config files instead of manual processes.

| IaC Tool | Type | Scope |
|----------|------|-------|
| **Terraform** | Declarative | Multi-cloud |
| CloudFormation | Declarative | AWS only |
| Ansible | Procedural | Config management |
| Pulumi | Imperative (code) | Multi-cloud |
| ARM Templates | Declarative | Azure only |

**CloudFormation vs Terraform:**

| Feature | CloudFormation | Terraform |
|---------|---------------|-----------|
| Developer | Amazon Web Services | HashiCorp |
| Cloud Support | AWS only | Multi-cloud |
| Language | JSON / YAML | HCL |
| State Management | Managed by AWS | `terraform.tfstate` |
| Multi-Cloud | ❌ | ✅ Strong support |
| Community | AWS community | Huge ecosystem |

---

### Q3. What is HCL (HashiCorp Configuration Language)?

HCL is Terraform's configuration language — human-readable and machine-parseable.

```hcl
# ────────────────────────────────────────────
# TERRAFORM HCL FUNDAMENTALS
# Language: HCL
# ────────────────────────────────────────────

# Provider block
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.5.0"
}

# Resource block
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name        = "web-server"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# Variable block
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Must be a valid t3 instance type."
  }
}

# Output block
output "instance_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web.public_ip
}

# Local block
locals {
  common_tags = {
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}
```

---

### Q4. What is a Terraform Provider?

A provider is a plugin that enables Terraform to interact with a specific cloud/service API.

```hcl
# ────────────────────────────────────────────
# PROVIDER CONFIGURATION
# Language: HCL
# ────────────────────────────────────────────

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"      # ~> means 5.x but not 6.x
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = ">= 2.0"
    }
    github = {
      source  = "integrations/github"
      version = "~> 5.0"
    }
  }
}

# Primary provider
provider "aws" {
  region  = "us-east-1"
  profile = "production"

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = var.environment
    }
  }
}

# Aliased provider (multiple regions)
provider "aws" {
  alias  = "eu"
  region = "eu-west-1"
}

resource "aws_instance" "eu_server" {
  provider      = aws.eu      # Use aliased provider
  ami           = "ami-..."
  instance_type = "t3.micro"
}
```

---

## 2. Core Commands

### Q5. What are the core Terraform commands?

```bash
#!/bin/bash
# ────────────────────────────────────────────
# CORE TERRAFORM COMMANDS
# Language: Shell (Bash)
# ────────────────────────────────────────────

# ── INITIALIZATION ──────────────────────────
terraform init                        # Initialize, download providers & modules
terraform init -upgrade               # Upgrade providers to latest matching version
terraform init -backend-config=b.cfg  # Use specific backend config
terraform init -reconfigure           # Reconfigure backend
terraform init -migrate-state         # Migrate state to new backend

# ── VALIDATION ──────────────────────────────
terraform validate                    # Check syntax and logic
terraform fmt                         # Format HCL files
terraform fmt -check                  # Check formatting (CI/CD use)
terraform fmt -recursive              # Format all subdirectories

# ── PLANNING ────────────────────────────────
terraform plan                        # Preview changes (dry run)
terraform plan -out=tfplan            # Save plan to file
terraform plan -var="env=prod"        # Override variable
terraform plan -refresh=false         # Skip state refresh (faster)
terraform plan -detailed-exitcode     # Exit code: 0=none, 1=error, 2=changes
terraform plan -target=aws_instance.web  # Plan specific resource

# ── APPLYING ────────────────────────────────
terraform apply                       # Apply changes (prompts)
terraform apply -auto-approve         # Apply without confirmation (CI/CD)
terraform apply tfplan                # Apply from saved plan
terraform apply -target=module.vpc    # Apply specific module
terraform apply -replace="aws_instance.web"  # Force replace resource
terraform apply -parallelism=20       # Increase concurrency (default 10)

# ── DESTROYING ──────────────────────────────
terraform destroy                     # Destroy all resources
terraform destroy -auto-approve       # No confirmation
terraform destroy -target=aws_instance.web  # Destroy specific resource

# ── INSPECTING ──────────────────────────────
terraform show                        # Show current state
terraform output                      # Show all outputs
terraform output vpc_id               # Show specific output
terraform output -json                # Machine-readable JSON
terraform output -raw dns_name        # Raw value (no quotes)
terraform console                     # Interactive expression evaluator
terraform graph                       # Dependency graph (DOT format)

# ── IMPORTING ───────────────────────────────
terraform import aws_instance.web i-1234567890abcdef0   # Import resource
terraform import aws_s3_bucket.data my-existing-bucket

# ── WORKSPACE ───────────────────────────────
terraform workspace list              # List workspaces
terraform workspace new dev           # Create + switch workspace
terraform workspace select prod       # Switch workspace
terraform workspace show              # Current workspace
terraform workspace delete dev        # Delete workspace
```

---

### Q6. What does `terraform init` do?

```bash
#!/bin/bash
terraform init
# What it does:
# 1. Downloads providers specified in required_providers
# 2. Downloads remote modules
# 3. Initializes backend (local or remote state)
# 4. Creates .terraform/ directory
# 5. Creates .terraform.lock.hcl (ALWAYS commit this!)
```

**Init output explained:**
```
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.31.0...
Terraform has been successfully initialized!
```

---

### Q7. What is `terraform plan` and why is it important?

```bash
#!/bin/bash
# Plan symbols:
# +   = will be CREATED
# -   = will be DESTROYED
# ~   = will be UPDATED in-place
# -/+ = will be DESTROYED and RECREATED
# <=  = data source will be READ

terraform plan

# Example output:
# Plan: 3 to add, 1 to change, 0 to destroy.
#
# + aws_instance.web          → will be created
# ~ aws_security_group.web    → will be updated
# -/+ aws_db_instance.main    → forces replacement

# Save plan for guaranteed same apply
terraform plan -out=my.tfplan
terraform apply my.tfplan
```

**Plan vs Apply:**

| | `terraform plan` | `terraform apply` |
|--|-----------------|------------------|
| Makes changes | ❌ Never | ✅ Yes |
| Output | Shows what WILL happen | Makes it happen |
| Use in CI | Always (PRs) | main branch only |
| Speed | Fast | Slower |

---

## 3. State Management

### Q8. What is Terraform State?

Terraform state is a JSON file (`terraform.tfstate`) mapping your config to real infrastructure.

```
What state tracks:
  ├── Resource IDs        (EC2 instance ID, RDS ARN)
  ├── Resource attributes (IP, endpoint, name)
  ├── Dependency graph    (what depends on what)
  └── Metadata            (provider version, resource type)
```

```bash
#!/bin/bash
# State commands
terraform show                          # View full state
terraform state list                    # List all resources
terraform state show aws_instance.web   # Show specific resource
cat terraform.tfstate                   # Raw JSON (never edit manually!)
```

⚠️ **State can contain passwords, private keys — protect with encryption + access controls!**

---

### Q11. What is Remote State and why is it important?

**Local vs Remote State:**

| Feature | Local State | Remote State |
|---------|------------|--------------|
| Storage | Local machine | S3/Azure Blob/GCS |
| Team collaboration | ❌ Difficult | ✅ Easy |
| State locking | ❌ No | ✅ Yes |
| Security | ❌ Low | ✅ Encrypted |
| Backup | ❌ Manual | ✅ Cloud versioning |
| Recommended | Small projects | Production |

```hcl
# ────────────────────────────────────────────
# REMOTE BACKEND CONFIGURATIONS
# Language: HCL
# ────────────────────────────────────────────

# AWS S3 + DynamoDB (most common)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"   # For state locking
  }
}

# Azure Blob Storage
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate12345"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Terraform Cloud
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

```hcl
# Create S3 + DynamoDB for state storage
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state"

  lifecycle {
    prevent_destroy = true   # Never delete state bucket!
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

### Q12. What is State Locking?

State locking prevents concurrent modifications that could corrupt state.

```
Without locking:                    With DynamoDB locking:
                                    
Engineer A → apply ─┐               Engineer A → apply → acquires lock ✅
                     ├─ CONFLICT!   Engineer B → apply → sees lock → waits ⏳
Engineer B → apply ─┘               Engineer A finishes → releases lock
                                    Engineer B → acquires lock → applies ✅
```

```bash
#!/bin/bash
# If a run crashed and left a stale lock
terraform force-unlock LOCK_ID    # Use ONLY when certain no one is running!

# Check DynamoDB for stale lock
aws dynamodb scan --table-name terraform-state-lock

# Manually delete stale lock
aws dynamodb delete-item \
  --table-name terraform-state-lock \
  --key '{"LockID": {"S": "s3://bucket/terraform.tfstate"}}'
```

---

### Q13. What is `terraform import`?

Brings existing manually-created infrastructure under Terraform management — without destroying it.

```bash
#!/bin/bash
# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0
terraform import aws_s3_bucket.data my-existing-bucket
terraform import aws_db_instance.main mydb-identifier
terraform import aws_security_group.web sg-0abc123def456789
```

**Process:**
```
1. Write resource block in .tf file
2. terraform import → maps to state
3. terraform plan   → should show no changes (if config matches)
4. Fix any drift between config and imported reality
```

```hcl
# Terraform 1.5+ — config-driven import (better!)
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}

resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}
```

```bash
terraform plan    # Shows import plan
terraform apply   # Imports + reconciles
```

---

### Q14. What is `terraform taint` / force replace?

```bash
#!/bin/bash
# Old way (deprecated in Terraform 1.0+)
terraform taint aws_instance.web
terraform untaint aws_instance.web

# Modern way — force replace
terraform apply -replace="aws_instance.web"
```

**Use cases:**
- EC2 instance became unresponsive
- Database got corrupted
- TLS certificate needs rotation
- Resource in error state needs recreation

---

### Q15. What is `terraform state mv`?

Rename or move resources without destroying them.

```bash
#!/bin/bash
# Rename resource
terraform state mv aws_instance.old_name aws_instance.new_name

# Move into module
terraform state mv aws_instance.web module.web_server.aws_instance.this

# Move out of module
terraform state mv module.web_server.aws_instance.this aws_instance.web

# Move to another state file
terraform state mv \
  -state=old.tfstate \
  -state-out=new.tfstate \
  aws_instance.web aws_instance.web
```

---

### Q16. `terraform destroy` vs `terraform state rm`

| Command | Cloud Resource | State |
|---------|---------------|-------|
| `terraform destroy` | ❌ DELETES it | Removes from state |
| `terraform state rm` | ✅ KEEPS it running | Removes from state only |

```bash
#!/bin/bash
# Actually delete the resource
terraform destroy -target=aws_instance.old_server

# Stop Terraform managing it (resource keeps running)
terraform state rm aws_instance.old_server
# Use when: handing off resource to another team/codebase
```

---

## 4. Variables & Outputs

### Q9. What are Terraform Variables?

```hcl
# ────────────────────────────────────────────
# VARIABLE TYPES
# Language: HCL
# ────────────────────────────────────────────

variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_count" {
  type    = number
  default = 2
}

variable "enable_monitoring" {
  type    = bool
  default = true
}

variable "allowed_cidrs" {
  type    = list(string)
  default = ["10.0.0.0/8", "172.16.0.0/12"]
}

variable "tags" {
  type = map(string)
  default = {
    Team    = "platform"
    Project = "infra"
  }
}

variable "db_config" {
  type = object({
    engine         = string
    instance_class = string
    storage        = number
  })
  sensitive = true    # Marks entire object as sensitive
}
```

**Variable precedence (highest → lowest):**
```
1. -var flag (CLI)              terraform apply -var="region=eu-west-1"
2. *.auto.tfvars file           prod.auto.tfvars (auto-loaded)
3. terraform.tfvars file        terraform.tfvars (auto-loaded)
4. TF_VAR_* env vars            export TF_VAR_region="eu-west-1"
5. Default value in variable block
```

---

### Q10. What are Terraform Outputs?

```hcl
# ────────────────────────────────────────────
# OUTPUTS
# Language: HCL
# ────────────────────────────────────────────

output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

output "db_endpoint" {
  description = "RDS endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true    # Hides in CLI output
}

output "instance_ips" {
  description = "All instance IPs"
  value       = [for inst in aws_instance.web : inst.public_ip]
}

output "load_balancer_dns" {
  value = aws_lb.main.dns_name
}
```

```bash
#!/bin/bash
terraform output                          # All outputs
terraform output vpc_id                   # Specific output
terraform output -json                    # Machine-readable
terraform output -raw load_balancer_dns   # Raw (no quotes)
```

---

## 5. Modules

### Q17. What is a Terraform Module?

A module is a container for multiple related resources — enabling reusability and organization.

```
infrastructure/
├── main.tf              ← Root module
├── variables.tf
├── outputs.tf
└── modules/
    ├── vpc/             ← Child module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── ec2/
    └── rds/
```

```hcl
# ────────────────────────────────────────────
# USING MODULES
# Language: HCL
# ────────────────────────────────────────────

# Local module
module "vpc" {
  source      = "./modules/vpc"
  cidr_block  = "10.0.0.0/16"
  environment = var.environment
}

# Terraform Registry module
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name = "my-cluster"
  vpc_id       = module.vpc.vpc_id
  subnet_ids   = module.vpc.private_subnet_ids
}

# Git module (pinned to version tag)
module "security" {
  source = "git::https://github.com/org/tf-modules.git//security?ref=v2.1.0"
  environment = var.environment
}

# Access module output
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnet_ids[0]
}
```

**Module sources:**

| Source | Example |
|--------|---------|
| Local path | `./modules/vpc` |
| Terraform Registry | `terraform-aws-modules/vpc/aws` |
| GitHub | `github.com/org/tf-modules//vpc` |
| Git + version | `git::https://...//vpc?ref=v1.5.0` |
| S3 | `s3::https://s3.amazonaws.com/bucket/vpc.zip` |

---

### Q18. Root module vs Child module

| | Root Module | Child Module |
|--|------------|--------------|
| Location | Directory where you run `terraform apply` | Called via `module` block |
| State | Direct state management | Part of root state |
| Providers | Defines providers | Inherits from caller |
| Inputs | Variables from .tfvars | Variables passed in `module` block |
| Outputs | Final outputs | Returned to calling module |

---

### Q19. How do you pass providers to child modules?

```hcl
# Root module
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "eu_west"
  region = "eu-west-1"
}

module "us_resources" {
  source = "./modules/app"
  # Uses default aws provider
}

module "eu_resources" {
  source = "./modules/app"
  providers = {
    aws = aws.eu_west    # Explicitly pass aliased provider
  }
}
```

---

## 6. Expressions & Functions

### Q21. What are `locals` in Terraform?

```hcl
# ────────────────────────────────────────────
# LOCALS
# Language: HCL
# ────────────────────────────────────────────

locals {
  environment = "production"
  app_name    = "my-app"

  # Computed tags (used everywhere)
  common_tags = {
    Environment = local.environment
    Application = local.app_name
    ManagedBy   = "Terraform"
    CostCenter  = "platform"
  }

  # Conditional value
  instance_type = var.environment == "production" ? "t3.medium" : "t3.micro"

  # Derived name
  bucket_name = "${var.project}-${var.environment}-${data.aws_region.current.name}"

  # Slice AZs
  azs = slice(data.aws_availability_zones.available.names, 0, 3)
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
  tags          = local.common_tags
}
```

---

### Q22. What are `count` and `for_each`?

```hcl
# ────────────────────────────────────────────
# COUNT — Integer-based repetition
# ────────────────────────────────────────────

resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-abc123"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"   # web-0, web-1, web-2
  }
}

# Conditional creation
resource "aws_cloudwatch_metric_alarm" "cpu" {
  count = var.enable_monitoring ? 1 : 0
  # ...
}

# Access: aws_instance.web[0]  aws_instance.web[1]
```

```hcl
# ────────────────────────────────────────────
# FOR_EACH — Map/Set-based repetition (preferred)
# ────────────────────────────────────────────

variable "users" {
  default = {
    alice = { role = "admin",  email = "alice@example.com" }
    bob   = { role = "viewer", email = "bob@example.com" }
  }
}

resource "aws_iam_user" "users" {
  for_each = var.users
  name     = each.key         # alice, bob
  tags     = { Email = each.value.email }
}

# for_each with set of strings
resource "aws_subnet" "public" {
  for_each          = toset(["us-east-1a", "us-east-1b", "us-east-1c"])
  vpc_id            = aws_vpc.main.id
  availability_zone = each.value
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, index(tolist(toset(["us-east-1a","us-east-1b","us-east-1c"])), each.value))
}

# Access: aws_iam_user.users["alice"]  aws_iam_user.users["bob"]
```

**count vs for_each:**

| | `count` | `for_each` |
|--|---------|-----------|
| Index type | Integer | String key |
| Remove middle item | Recreates ALL after it ⚠️ | Only removes that item ✅ |
| Best for | Identical resources | Resources with unique configs |

---

### Q24. Common Terraform Functions

```hcl
# ────────────────────────────────────────────
# BUILT-IN FUNCTIONS
# Language: HCL
# ────────────────────────────────────────────

# String
upper("hello")                    # "HELLO"
lower("HELLO")                    # "hello"
format("app-%s", var.env)         # "app-production"
replace("hello world", "world", "tf")  # "hello tf"
split(",", "a,b,c")               # ["a", "b", "c"]
join("-", ["a", "b", "c"])        # "a-b-c"
trimspace("  hello  ")            # "hello"
contains(["a","b"], "a")          # true

# Numbers
max(1, 2, 3)                      # 3
min(1, 2, 3)                      # 1
ceil(1.2)                         # 2
floor(1.9)                        # 1

# Collections
length(["a","b","c"])             # 3
toset(["a","b","a"])              # {"a","b"} — dedup
flatten([[1,2],[3,4]])             # [1,2,3,4]
merge({a=1}, {b=2})               # {a=1, b=2}
keys({a=1, b=2})                  # ["a","b"]
values({a=1, b=2})                # [1, 2]
lookup({a=1, b=2}, "a", 0)       # 1 (with default)
compact(["a","","b"])             # ["a","b"] — removes empty
distinct(["a","b","a"])           # ["a","b"]

# Networking
cidrsubnet("10.0.0.0/16", 8, 1)  # "10.0.1.0/24"
cidrhost("10.0.0.0/24", 5)       # "10.0.0.5"

# Encoding
base64encode("hello")             # "aGVsbG8="
jsonencode({a=1})                 # "{\"a\":1}"
jsondecode("{\"a\":1}")           # {a=1}

# Files
file("./user_data.sh")            # Read file
templatefile("./tmpl.tpl", {name = "world"})  # Render template
filebase64("./cert.pem")          # File as base64

# Type conversion
tostring(42)                      # "42"
tonumber("42")                    # 42
tobool("true")                    # true
```

---

### Q25. What is a `data` source?

Data sources READ existing infrastructure — they don't create anything.

```hcl
# ────────────────────────────────────────────
# DATA SOURCES
# Language: HCL
# ────────────────────────────────────────────

# Latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Existing VPC
data "aws_vpc" "existing" {
  tags = { Name = "production-vpc" }
}

# Current account + region
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# Available AZs
data "aws_availability_zones" "available" {
  state = "available"
}

# SSM Parameter
data "aws_ssm_parameter" "db_password" {
  name = "/prod/db/password"
}

# Use in resources
resource "aws_instance" "web" {
  ami               = data.aws_ami.amazon_linux.id
  instance_type     = "t3.micro"
  availability_zone = data.aws_availability_zones.available.names[0]

  tags = {
    AccountID = data.aws_caller_identity.current.account_id
    Region    = data.aws_region.current.name
  }
}
```

---

## 7. Workspaces & Environments

### Q26. What are Terraform Workspaces?

```bash
#!/bin/bash
# Workspace commands
terraform workspace list          # List all (default always exists)
terraform workspace new dev       # Create + switch to dev
terraform workspace select prod   # Switch to prod
terraform workspace show          # Current workspace
terraform workspace delete dev    # Delete (must be empty)
```

```hcl
# Use workspace name in config
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "production" ? "t3.medium" : "t3.micro"
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}
```

---

### Q27. Strategies for managing multiple environments

**Architecture diagram:**
```
Strategy 1: Workspaces (shared code, different state)
  terraform.tfstate          (default)
  env:/dev/terraform.tfstate
  env:/staging/terraform.tfstate
  env:/prod/terraform.tfstate

Strategy 2: Directory-based (RECOMMENDED for production)
  infra/
  ├── modules/           ← Shared modules
  │   ├── vpc/
  │   └── eks/
  └── environments/
      ├── dev/           ← Separate state, separate vars
      │   ├── backend.tf
      │   └── terraform.tfvars
      ├── staging/
      └── production/

Strategy 3: Terragrunt (DRY)
  live/
  ├── terragrunt.hcl     ← Root config (backend, providers)
  ├── dev/
  │   └── vpc/terragrunt.hcl
  └── prod/
      └── vpc/terragrunt.hcl
```

---

## 8. Advanced Concepts

### Q28. What is `depends_on`?

```hcl
# ────────────────────────────────────────────
# DEPENDS_ON — Explicit dependencies
# Language: HCL
# ────────────────────────────────────────────

resource "aws_iam_role_policy" "example" {
  role   = aws_iam_role.example.id
  policy = data.aws_iam_policy_document.example.json
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.app.id
  instance_type = "t3.micro"

  # Not referenced by attribute, but must exist first
  depends_on = [aws_iam_role_policy.example]
}

# Module-level dependency
module "ecs_service" {
  source     = "./modules/ecs"
  depends_on = [module.vpc, aws_iam_role.ecs_task]
}
```

---

### Q29. What is `lifecycle`?

```hcl
# ────────────────────────────────────────────
# LIFECYCLE RULES
# Language: HCL
# ────────────────────────────────────────────

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  lifecycle {
    # Create NEW before destroying OLD (zero-downtime)
    create_before_destroy = true

    # Ignore external changes (managed outside Terraform)
    ignore_changes = [
      tags["LastUpdated"],
      user_data,     # Don't replace if user_data changes
      ami            # Handle AMI updates via ASG instead
    ]

    # Block accidental deletion
    prevent_destroy = true

    # Custom validation (Terraform 1.2+)
    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "AMI must be x86_64."
    }

    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP."
    }
  }
}
```

---

### Q30. What is `templatefile()`?

```hcl
# Template: templates/user_data.sh.tpl
# #!/bin/bash
# docker run -d \
#   -e DB_HOST=${db_host} \
#   -e APP_ENV=${environment} \
#   ${docker_image}

resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  user_data = templatefile("${path.module}/templates/user_data.sh.tpl", {
    db_host      = aws_db_instance.main.endpoint
    environment  = var.environment
    docker_image = "myapp:${var.app_version}"
  })
}
```

---

### Q31. What is `dynamic` block?

```hcl
# ────────────────────────────────────────────
# DYNAMIC BLOCKS — Generate repeated nested blocks
# Language: HCL
# ────────────────────────────────────────────

variable "ingress_rules" {
  default = [
    { port = 80,  protocol = "tcp", cidr = "0.0.0.0/0",  description = "HTTP" },
    { port = 443, protocol = "tcp", cidr = "0.0.0.0/0",  description = "HTTPS" },
    { port = 22,  protocol = "tcp", cidr = "10.0.0.0/8", description = "SSH" },
  ]
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = [ingress.value.cidr]
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### Q33. What is `moved` block?

```hcl
# ────────────────────────────────────────────
# MOVED BLOCK — Rename/move without destroying
# Language: HCL (Terraform 1.1+)
# ────────────────────────────────────────────

# Renamed resource
moved {
  from = aws_instance.server
  to   = aws_instance.web_server
}

# Moved into module
moved {
  from = aws_instance.web
  to   = module.web_server.aws_instance.this
}

# Moved between modules
moved {
  from = module.old_module.aws_s3_bucket.data
  to   = module.new_module.aws_s3_bucket.data
}
```

---

### Q34. What is Terraform `check` block?

```hcl
# ────────────────────────────────────────────
# CHECK BLOCKS — Post-apply assertions (Terraform 1.5+)
# Language: HCL
# ────────────────────────────────────────────

check "health_check" {
  data "http" "app_health" {
    url = "https://${aws_lb.main.dns_name}/health"
  }

  assert {
    condition     = data.http.app_health.status_code == 200
    error_message = "App health check failed: ${data.http.app_health.status_code}"
  }
}

check "certificate_expiry" {
  assert {
    condition     = timecmp(data.tls_certificate.app.not_after,
                            timeadd(timestamp(), "720h")) > 0
    error_message = "TLS cert expires in less than 30 days!"
  }
}
```

---

### Q32. What is `terraform_remote_state` data source?

```hcl
# ────────────────────────────────────────────
# REMOTE STATE — Share outputs between stacks
# Language: HCL
# ────────────────────────────────────────────

# Networking stack outputs:
# output "vpc_id" { value = aws_vpc.main.id }

# Application stack reads networking outputs:
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.networking.outputs.private_subnet_ids[0]
}
```

---

## 9. Security

### Q36. What is `sensitive` in Terraform?

```hcl
variable "db_password" {
  type      = string
  sensitive = true    # Hidden in plan output + logs
}

locals {
  # Auto-sensitive because it references sensitive value
  connection_string = "postgresql://admin:${var.db_password}@${aws_db_instance.main.endpoint}/db"
}

output "db_connection_string" {
  value     = local.connection_string
  sensitive = true    # Required for sensitive outputs
}
```

---

### Secrets Management Best Practices

```hcl
# ────────────────────────────────────────────
# SECRETS — Never hardcode!
# Language: HCL
# ────────────────────────────────────────────

# ❌ BAD
resource "aws_db_instance" "main" {
  password = "hardcoded-password"   # NEVER do this!
}

# ✅ OPTION 1: Environment variable
# export TF_VAR_db_password="$(aws secretsmanager get-secret-value ...)"
variable "db_password" {
  type      = string
  sensitive = true
}

# ✅ OPTION 2: Read from AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db.secret_string)["password"]
}

# ✅ OPTION 3: Generate random + store in Secrets Manager (BEST)
resource "random_password" "db" {
  length  = 32
  special = false
}

resource "aws_secretsmanager_secret_version" "db_pass" {
  secret_id     = aws_secretsmanager_secret.db_pass.id
  secret_string = random_password.db.result
}

resource "aws_db_instance" "main" {
  password = random_password.db.result
}

# ✅ OPTION 4: HashiCorp Vault
data "vault_generic_secret" "db_creds" {
  path = "secret/prod/db"
}

resource "aws_db_instance" "main" {
  password = data.vault_generic_secret.db_creds.data["password"]
}
```

---

## 10. CI/CD Integration

### Terraform in GitHub Actions

```yaml
# ────────────────────────────────────────────
# GITHUB ACTIONS — Terraform Pipeline
# Language: YAML
# ────────────────────────────────────────────

name: Terraform
on:
  pull_request:
    paths: ['infrastructure/**']
  push:
    branches: [main]
    paths: ['infrastructure/**']

jobs:
  terraform:
    name: Terraform
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: infrastructure/environments/production

    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/TerraformRole
          aws-region: us-east-1

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan (PR only)
        if: github.event_name == 'pull_request'
        run: terraform plan -no-color -out=tfplan

      - name: Terraform Apply (main only)
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve tfplan
```

---

### Terraform Testing

```bash
#!/bin/bash
# ────────────────────────────────────────────
# TERRAFORM TESTING LEVELS
# Language: Shell (Bash)
# ────────────────────────────────────────────

# Level 1: Static analysis
terraform fmt -check          # Formatting
terraform validate            # Syntax/logic
tflint                        # Provider-specific lint
checkov -d .                  # Security + compliance
tfsec .                       # Security scanning
infracost breakdown --path .  # Cost estimation

# Level 2: Plan testing
terraform plan -detailed-exitcode
# Exit 0 = no changes
# Exit 1 = error
# Exit 2 = changes present

# Level 3: Native testing (Terraform 1.6+)
# tests/vpc.tftest.hcl
terraform test
```

```hcl
# Terraform native test file
# tests/vpc.tftest.hcl

run "vpc_is_created" {
  command = plan

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block is incorrect"
  }

  assert {
    condition     = aws_vpc.main.enable_dns_hostnames == true
    error_message = "DNS hostnames should be enabled"
  }
}

run "subnets_across_azs" {
  command = apply

  assert {
    condition     = length(aws_subnet.public) == 3
    error_message = "Should have 3 public subnets"
  }
}
```

---

## 11. Scenario-Based Questions

### Scenario 1: `terraform apply` failed halfway — partial infrastructure created

```bash
#!/bin/bash
# Resources created BEFORE failure ARE in state — Terraform is idempotent

# Step 1: Read error carefully
terraform plan   # Shows remaining changes

# Step 2: Fix the issue (wrong AMI, insufficient permissions, quota)

# Step 3: Re-run — Terraform picks up from where it left off
terraform apply
# Already-created resources won't be recreated!
```

---

### Scenario 2: Rename a resource without destroying it

```hcl
# OLD: resource "aws_instance" "server" { ... }
# NEW: resource "aws_instance" "web_server" { ... }
# Without moved block: DESTROYS server + CREATES web_server!

# FIX: Add moved block
moved {
  from = aws_instance.server
  to   = aws_instance.web_server
}
```

```bash
#!/bin/bash
# Alternative: state mv
terraform state mv aws_instance.server aws_instance.web_server
terraform plan   # Should show: No changes. Infrastructure is up-to-date.
```

---

### Scenario 3: Bring manually created resource under Terraform

```bash
#!/bin/bash
# Step 1: Write resource block
# resource "aws_s3_bucket" "logs" { bucket = "my-existing-bucket" }

# Step 2: Import
terraform import aws_s3_bucket.logs my-existing-logs-bucket

# Step 3: Check drift
terraform plan   # Fix .tf config if there are differences

# Step 4: Reconcile
terraform apply
```

---

### Scenario 4: State lock — no one is running Terraform

```bash
#!/bin/bash
# Step 1: Verify NO ONE is actually running
# Check CI/CD pipeline, ask team

# Step 2: Get lock ID
terraform plan
# Error: Lock Info: ID: abc-123-xyz

# Step 3: Force unlock (ONLY when certain it's stale!)
terraform force-unlock abc-123-xyz

# Step 4: Manual cleanup if needed
aws dynamodb delete-item \
  --table-name terraform-state-lock \
  --key '{"LockID": {"S": "s3://bucket/terraform.tfstate"}}'
```

---

### Scenario 5: Upgrade provider versions safely

```bash
#!/bin/bash
# Step 1: Check current versions
cat .terraform.lock.hcl

# Step 2: Update version constraint
# ~> 4.0  →  ~> 5.0

# Step 3: Check provider changelog for breaking changes

# Step 4: Upgrade lock file
terraform init -upgrade

# Step 5: Test in non-prod first
terraform plan
terraform apply

# Step 6: Commit lock file
git add .terraform.lock.hcl
git commit -m "chore: upgrade AWS provider to 5.x"
```

---

### Scenario 6: Large team — manage Terraform safely

```bash
#!/bin/bash
# Structure
# infrastructure/
# ├── modules/           (shared, versioned)
# ├── environments/
# │   ├── dev/           (separate backend key)
# │   ├── staging/
# │   └── production/    (protected workspace)
# └── .github/workflows/

# Rules:
# 1. ALL changes via PR → auto terraform plan comment
# 2. main merge → auto terraform apply (prod: manual approval)
# 3. Use OIDC for AWS auth (no long-lived keys)
# 4. Remote state with DynamoDB locking
# 5. Sentinel/OPA policies for guardrails
```

---

### Scenario 7: Prevent accidental destruction of critical resources

```hcl
# Method 1: prevent_destroy lifecycle
resource "aws_db_instance" "production" {
  lifecycle {
    prevent_destroy = true   # Terraform errors on destroy attempt
  }
}

# Method 2: Separate state for critical resources
# database/ has its own backend — even if app state is destroyed, DB is safe

# Method 3: IAM restrictions
# CI/CD role has no DeleteDB* permissions in production
```

---

### Scenario 8: Plan shows unexpected -/+ (replacement)

```bash
#!/bin/bash
# Read plan output: "forces replacement" keyword
# -/+ aws_instance.web (must be replaced)
#     ~ ami: "ami-old" → "ami-new"  (forces replacement)
```

```hcl
# Fix: lifecycle ignore_changes
resource "aws_instance" "web" {
  lifecycle {
    ignore_changes        = [ami]                  # Don't replace on AMI change
    create_before_destroy = true                   # If replacement needed: no downtime
  }
}
```

---

### Scenario 9: Zero-downtime deployment with Terraform

```hcl
# Blue-Green with ALB weighted routing
resource "aws_lb_listener_rule" "main" {
  listener_arn = aws_lb_listener.https.arn
  action {
    type = "forward"
    forward {
      target_group {
        arn    = aws_lb_target_group.blue.arn
        weight = var.blue_weight   # 100 → 0 during cutover
      }
      target_group {
        arn    = aws_lb_target_group.green.arn
        weight = var.green_weight  # 0 → 100 during cutover
      }
    }
  }
}
# Switch: terraform apply -var="blue_weight=0" -var="green_weight=100"
```

---

### Scenario 10: State file too large — split monolith

```bash
#!/bin/bash
# Split into: networking.tfstate + database.tfstate + app-a.tfstate

# Move resources to new state files
terraform state mv \
  -state=old.tfstate \
  -state-out=networking/terraform.tfstate \
  aws_vpc.main aws_vpc.main

terraform state mv \
  -state=old.tfstate \
  -state-out=networking/terraform.tfstate \
  'aws_subnet.private[0]' 'aws_subnet.private[0]'

# Share values between states using remote_state
# terraform plan in each new directory → should show no changes
```

---

### Scenario 11: Terraform running across 10 AWS accounts

```hcl
# Separate backend per account
terraform {
  backend "s3" {
    bucket   = "tf-state-${account_id}"
    key      = "global/terraform.tfstate"
    region   = "us-east-1"
    role_arn = "arn:aws:iam::${account_id}:role/TerraformRole"
  }
}
```

```bash
#!/bin/bash
# Terragrunt for DRY multi-account
terragrunt run-all plan    # Plan all accounts
terragrunt run-all apply   # Apply all accounts
```

---

### Scenario 12: Terraform apply is very slow

```bash
#!/bin/bash
# 1. Skip refresh if state is trusted
terraform apply -refresh=false

# 2. Target specific resources
terraform apply -target=module.application

# 3. Increase parallelism (default: 10)
terraform apply -parallelism=20

# 4. Split state into smaller stacks

# 5. Measure what's slow
TF_LOG=TRACE terraform plan 2>&1 | grep "time="
```

---

### Scenario 13: Terraform drift — infrastructure changed outside Terraform

```bash
#!/bin/bash
# Detect drift
terraform plan   # Shows what changed outside Terraform

# Refresh state to match reality (without making changes)
terraform apply -refresh-only

# Option A: Let Terraform fix drift
terraform apply   # Reverts manual changes

# Option B: Accept drift — update .tf files to match manual changes
# Edit .tf → terraform plan → should show no changes

# Prevent future drift
# - IAM policies restricting manual changes
# - AWS Config alerts on drift
# - Regular terraform plan in CI/CD (drift detection)
```

---

### Scenario 14: Secrets exposed in state file

```bash
#!/bin/bash
# 1. IMMEDIATELY rotate all exposed credentials

# 2. Encrypt backend
# Ensure S3 bucket has encryption enabled

# 3. Restrict access to state file
aws s3api put-bucket-policy \
  --bucket my-terraform-state \
  --policy file://restrict-policy.json

# 4. Use sensitive = true for variables

# 5. Use Secrets Manager instead of storing in state
# random_password → Secrets Manager → aws_db_instance (reference SM)
```

---

### Scenario 15: Testing Terraform before production

```bash
#!/bin/bash
# Full testing pipeline:

# 1. Format check
terraform fmt -check -recursive

# 2. Validate
terraform validate

# 3. Security scan
tfsec .
checkov -d .

# 4. Cost estimate
infracost breakdown --path .

# 5. Plan review (human approval for prod)
terraform plan -out=tfplan

# 6. Terratest (automated)
go test ./tests/ -v -timeout 30m

# 7. Apply to staging first
terraform apply tfplan

# 8. Smoke test staging
curl https://staging.app.com/health

# 9. Promote to production (separate approval)
terraform apply -auto-approve   # In prod pipeline
```

---

### Scenario 16: Handle Terraform for microservices — multiple stacks

```
Recommended stacks:
├── networking/    → VPC, subnets, IGW (rarely changes)
├── security/      → IAM, security groups
├── database/      → RDS, ElastiCache (sensitive, separate)
├── services/
│   ├── auth/      → Auth service
│   ├── api/       → API service
│   └── frontend/  → Frontend
└── monitoring/    → CloudWatch, Grafana
```

```hcl
# Share data between stacks
data "terraform_remote_state" "networking" {
  backend = "s3"
  config  = { bucket = "tf-state", key = "networking/terraform.tfstate", region = "us-east-1" }
}

resource "aws_ecs_service" "auth" {
  network_configuration {
    subnets = data.terraform_remote_state.networking.outputs.private_subnet_ids
  }
}
```

---

### Scenario 17: Cost control with Terraform

```hcl
# Different sizes per environment
variable "instance_type" {
  default = {
    dev        = "t3.micro"
    staging    = "t3.small"
    production = "t3.medium"
  }
}

resource "aws_instance" "web" {
  instance_type = var.instance_type[var.environment]
}

# Auto-shutdown non-prod at night
resource "aws_autoscaling_schedule" "scale_down" {
  count                  = var.environment != "production" ? 1 : 0
  scheduled_action_name  = "scale-down-evenings"
  min_size               = 0
  max_size               = 0
  desired_capacity       = 0
  recurrence             = "0 20 * * 1-5"   # 8PM Mon-Fri
  autoscaling_group_name = aws_autoscaling_group.web.name
}

# S3 lifecycle — delete old logs
resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id
  rule {
    id     = "expire-logs"
    status = "Enabled"
    expiration { days = 90 }
  }
}
```

---

### Scenario 18: Refactor Terraform without downtime

```bash
#!/bin/bash
# Extract inline resource into module — without destroying it

# Before: resource "aws_security_group" "web" { ... }
# After:  module "web_sg" { source = "./modules/sg" }
```

```hcl
# Add moved block
moved {
  from = aws_security_group.web
  to   = module.web_sg.aws_security_group.this
}
```

```bash
#!/bin/bash
terraform plan    # Shows: No changes (moved block worked)
terraform apply   # Only updates state, no resource recreation
# After team pulls latest: remove moved block
```

---

### Scenario 19: Plan works locally but fails in CI/CD pipeline

```bash
#!/bin/bash
# Common causes:
# 1. Missing credentials in pipeline
#    → Add AWS role assumption (OIDC)
# 2. Different Terraform version
#    → Pin version in setup-terraform action
# 3. Missing TF_VAR_ env vars
#    → Add to pipeline secrets
# 4. Backend not accessible
#    → Check VPC, S3 bucket policy
# 5. .terraform.lock.hcl not committed
#    → git add .terraform.lock.hcl

# Debug
TF_LOG=DEBUG terraform plan 2>&1 | head -100
```

---

### Scenario 20: Wrong workspace selected — production hit

```bash
#!/bin/bash
# Prevention:
terraform workspace show   # Always verify before apply!

# Add this to your shell profile
# PS1="\$(terraform workspace show 2>/dev/null) $ "

# CI/CD: Always explicitly select workspace
terraform workspace select production
terraform apply

# Use separate backend keys per environment
# Never rely on workspace name alone — use backend key too
```

---

### Scenarios 21–55: Additional Quick Scenarios

**21. Provider plugin error:**
```bash
terraform init -upgrade    # Reinitialize + upgrade providers
rm -rf .terraform/         # Clear plugin cache
terraform init             # Fresh download
```

**22. Infrastructure not consistent across envs:**
```bash
# Use same module with different tfvars
# Audit with: terraform plan -var-file=envs/dev.tfvars
# Drift detection: terraform plan -refresh-only
```

**23. Module working in staging, failing in production:**
```bash
# Check: IAM permissions (prod has stricter)
# Check: Variables (diff values in prod.tfvars)
# Check: Resource limits/quotas (prod account)
# Check: Provider version pinning
```

**24. Resource recreated unexpectedly:**
```hcl
# Immutable field changed → must recreate
# Fix: lifecycle { create_before_destroy = true }
# Check: Avoid name changes on immutable resources
# Check: Use bucket_prefix instead of bucket for S3
```

**25. Variables not passed correctly:**
```bash
# Check precedence (CLI > auto.tfvars > .tfvars > env > default)
terraform plan -var-file="prod.tfvars"
terraform console   # Test: var.instance_type
```

**26. Output not visible:**
```bash
terraform output   # Run after apply
# If empty: check output block exists + terraform apply ran
# If hidden: add -json flag for sensitive outputs
terraform output -json   # Shows sensitive values
```

**27. Version mismatch in team:**
```hcl
terraform {
  required_version = ">= 1.5.0, < 2.0.0"
}
# Use tfenv or asdf to manage versions
# Pin in CI/CD: hashicorp/setup-terraform@v3 with terraform_version
```

**28. Destroy specific resources:**
```bash
terraform destroy -target=aws_instance.old_server
terraform destroy -target=module.legacy
```

**29. Dependency cycle detected:**
```bash
terraform graph | dot -Tpng > graph.png   # Visualize dependency graph
# Remove circular reference
# Use data source instead of resource reference
```

**30. Backend migration failed midway:**
```bash
terraform init -migrate-state   # Retry migration
# If corrupted: restore from S3 versioning
aws s3api list-object-versions \
  --bucket my-tf-state --prefix terraform.tfstate
aws s3api get-object \
  --bucket my-tf-state \
  --version-id <version-id> \
  terraform.tfstate
```

**31. Terraform apply removed critical resources:**
```bash
# Root cause: No prevent_destroy + no approval gate
# Immediate: velero/backup restore or re-import
terraform import aws_db_instance.main mydb-id
# Prevention: prevent_destroy = true on critical resources
```

**32. CI/CD pipeline using outdated state:**
```bash
terraform init -reconfigure   # Force re-init backend
terraform apply -refresh=true  # Force state refresh
# Check: pipeline caches .terraform/ incorrectly
```

**33. Kubernetes resources not updating:**
```bash
# K8s provider drift — apply -refresh-only first
terraform apply -refresh-only
# Check: K8s resource immutable fields changed
# Check: kubeconfig path correct in CI/CD
```

**34. Autoscaling deployed but not scaling:**
```hcl
# Check: metrics-server installed
# Check: IAM permissions for autoscaling:*
# Check: Target tracking policy configured
# Check: Min/Max values allow scaling
```

**35. Multiple users, same state corruption:**
```bash
# Solution: Remote state + DynamoDB locking
# Rule: NEVER run terraform apply manually in production
# Use: CI/CD pipeline with approval gate
# Monitor: CloudWatch alarm on concurrent DynamoDB lock attempts
```

**36. Provider version mismatch broke deployment:**
```bash
# Prevention: Pin exact versions in required_providers
# version = "= 5.31.0"   # Exact pin
# version = "~> 5.31"    # Minor range pin
git add .terraform.lock.hcl   # ALWAYS commit lock file!
```

**37. State corrupted during deployment:**
```bash
# Restore from S3 versioning (enable versioning on state bucket!)
aws s3 cp \
  s3://my-state-bucket/prod/terraform.tfstate \
  ./terraform.tfstate \
  --version-id <previous-version-id>
terraform plan   # Verify restored state
```

**38. Secrets rotation — update without downtime:**
```hcl
# Use lifecycle ignore_changes for password
resource "aws_db_instance" "main" {
  password = var.db_password
  lifecycle {
    ignore_changes = [password]   # Terraform won't change after initial
  }
}
# Rotate via: aws secretsmanager rotate-secret
```

**39. Cross-account resource reference:**
```hcl
# Use assume_role in provider
provider "aws" {
  alias = "prod_account"
  assume_role {
    role_arn = "arn:aws:iam::PROD_ACCOUNT_ID:role/TerraformRole"
  }
}
```

**40. Remote backend inaccessible during deploy:**
```bash
# Terraform CANNOT continue — it needs state to know current infra
# Fix:
# 1. Check S3 bucket policy
# 2. Check IAM permissions
# 3. Check VPC endpoints if in private subnet
# 4. Check DynamoDB table exists (for locking)
```

**41. Canary deployment with Terraform:**
```hcl
# weighted routing on ALB
forward {
  target_group { arn = aws_lb_target_group.v1.arn; weight = 95 }
  target_group { arn = aws_lb_target_group.v2.arn; weight = 5  }
}
# Gradually: weight = 50/50, then 0/100
```

**42. Module not working — check list:**
```bash
terraform init           # Re-download module
terraform validate       # Check module inputs/outputs
terraform console        # Test: module.vpc.vpc_id
# Check: Module version compatibility
# Check: Required variables all provided
# Check: Provider passed correctly
```

**43. Terraform plan shows resource deleted — accidentally:**
```bash
# STOP! Don't apply. Investigate first.
terraform state show <resource>   # Verify it's in state
# Common cause: Resource moved to module (needs moved block)
# Common cause: for_each key changed
# Common cause: Variable changed that affects for_each
```

**44. Infrastructure changes not picked up:**
```bash
terraform refresh         # Sync state with actual infrastructure
terraform apply -refresh-only  # Modern equivalent
# Then plan will show actual changes
```

**45. Plan works, apply fails with permission error:**
```bash
# Plan uses ReadOnly, Apply uses Write permissions
# Add missing permissions to Terraform IAM role:
# ec2:CreateTags, rds:ModifyDBInstance, etc.
# Check: Service Control Policies (SCPs) in AWS Organizations
```

**46. Resource created but wrong AZ:**
```hcl
resource "aws_instance" "web" {
  availability_zone = var.availability_zone   # Explicitly specify
  subnet_id         = aws_subnet.private[var.az_index].id
}
```

**47. State shows resource but it doesn't exist in AWS:**
```bash
# Orphaned state entry
terraform state rm aws_instance.orphaned
# OR: Re-import if resource was recreated
terraform import aws_instance.web i-newid
```

**48. Terraform validate passes but plan fails:**
```bash
# validate only checks syntax — not API responses
# Common causes:
# - AMI doesn't exist in that region
# - Instance type not available in AZ
# - S3 bucket name already taken
# - Quota exceeded
```

**49. Module version pinning best practice:**
```hcl
# ALWAYS pin modules in production
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "= 5.1.2"   # Exact pin for production
  # version = "~> 5.1"  # Minor range for dev
}
# Update: change version → terraform init -upgrade
```

**50. Terraform test automation:**
```bash
#!/bin/bash
# Full CI/CD test pipeline
set -e

terraform fmt -check -recursive
terraform validate
tflint --recursive
tfsec .
checkov -d . --compact
infracost breakdown --path . --format table
terraform plan -out=tfplan -detailed-exitcode
# If exit code 2 (changes) → require approval
# terraform apply tfplan (after approval)
```

**51. Import block for bulk import (Terraform 1.5+):**
```hcl
# Generate import blocks for all existing EC2 instances
import {
  for_each = {
    web = "i-0abc123def456789"
    api = "i-0def456abc789123"
    db  = "i-0789abc123def456"
  }
  to = aws_instance.servers[each.key]
  id = each.value
}
```

**52. Terragrunt for DRY code:**
```hcl
# root terragrunt.hcl
remote_state {
  backend = "s3"
  config = {
    bucket  = "tf-state-${get_aws_account_id()}"
    key     = "${path_relative_to_include()}/terraform.tfstate"
    region  = "us-east-1"
    encrypt = true
  }
}

# dev/vpc/terragrunt.hcl
terraform {
  source = "git::https://github.com/org/modules.git//vpc?ref=v2.0"
}

inputs = {
  cidr_block  = "10.0.0.0/16"
  environment = "dev"
}
```

**53. OPA/Sentinel policy as code:**
```hcl
# Sentinel policy — no internet-facing RDS
main = rule {
  all tfplan.resource_changes as _, resource_change {
    resource_change.type is not "aws_db_instance" or
    resource_change.change.after.publicly_accessible is false
  }
}
```

**54. Terraform drift auto-detection in CI:**
```yaml
# .github/workflows/drift-detection.yml
on:
  schedule:
    - cron: "0 */6 * * *"   # Every 6 hours

jobs:
  drift:
    steps:
      - run: terraform plan -detailed-exitcode
      # Exit 2 = drift detected → create GitHub issue or PagerDuty alert
```

**55. Full debug approach for any Terraform issue:**
```bash
#!/bin/bash
# Step-by-step debug
echo "1. Check error logs"
terraform plan 2>&1 | head -50

echo "2. Enable debug logging"
TF_LOG=DEBUG terraform plan 2>&1 | grep -E "error|warn|Error"

echo "3. Validate config"
terraform validate

echo "4. Check state"
terraform state list
terraform show

echo "5. Verify provider auth"
aws sts get-caller-identity

echo "6. Check backend"
terraform init -reconfigure

echo "7. Refresh state"
terraform apply -refresh-only

echo "8. Test in staging"
terraform workspace select staging
terraform plan
```

---

## 12. Quick Reference

```bash
#!/bin/bash
# ────────────────────────────────────────────
# TERRAFORM COMMANDS CHEATSHEET
# Language: Shell (Bash)
# ────────────────────────────────────────────

# SETUP
terraform init                      # Initialize
terraform init -upgrade             # Upgrade providers
terraform init -migrate-state       # Migrate backend

# VALIDATION
terraform validate                  # Syntax check
terraform fmt -recursive            # Format all files
terraform fmt -check                # Check formatting (CI)

# PLANNING
terraform plan                      # Preview changes
terraform plan -out=plan.tfplan     # Save plan
terraform plan -refresh=false       # Skip refresh
terraform plan -target=module.vpc   # Target specific

# APPLYING
terraform apply                     # Apply (with confirm)
terraform apply -auto-approve       # No confirm (CI/CD)
terraform apply plan.tfplan         # Apply saved plan
terraform apply -replace="resource" # Force replace
terraform apply -parallelism=20     # More concurrency

# STATE
terraform state list                # List resources
terraform state show <resource>     # Show resource
terraform state mv <from> <to>      # Move/rename
terraform state rm <resource>       # Remove from state
terraform state pull                # Download remote state
terraform state push                # Upload state

# IMPORT
terraform import <resource> <id>    # Import existing resource

# WORKSPACE
terraform workspace list            # List workspaces
terraform workspace new <name>      # Create workspace
terraform workspace select <name>   # Switch workspace
terraform workspace show            # Current workspace

# DESTROY
terraform destroy                   # Destroy all
terraform destroy -target=<res>     # Destroy specific

# UTILS
terraform output                    # Show outputs
terraform output -json              # JSON outputs
terraform console                   # REPL
terraform graph                     # Dependency graph
terraform force-unlock <id>         # Unlock state
terraform providers                 # Show providers
terraform version                   # Show version
```

**Terraform Best Practices:**

| Practice | Why |
|----------|-----|
| ✅ Remote backend always | Team collaboration + locking |
| ✅ Commit `.terraform.lock.hcl` | Reproducible provider versions |
| ✅ Pin module versions | No surprise breaking changes |
| ✅ `prevent_destroy` on critical | No accidental deletion |
| ✅ `sensitive = true` for secrets | No leaks in logs |
| ✅ Use `moved` blocks | Refactor without destroying |
| ✅ Separate state per environment | Blast radius isolation |
| ✅ Plan before every apply | No surprises |
| ✅ Tag everything | Cost visibility |
| ❌ Never hardcode secrets | Use Vault/Secrets Manager |
| ❌ Never edit state manually | Use state commands |
| ❌ Never force-unlock in use | Causes corruption |
| ❌ Never `latest` for modules in prod | Use pinned versions |

---

