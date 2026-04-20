# What is Terraform?
Terraform is an **open-source Infrastructure as Code (IaC)** tool developed by HashiCorp.
It allows you to **define, provision, and manage infrastructure** across multiple cloud providers using a **declarative language** called **HCL (HashiCorp Configuration Language).**

# Terraform Fundamentals
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/856cf5ea-46ef-4436-8fa6-a355f6bc873b" />

# Terraform Works
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/a780375b-572f-4dee-a968-12a55fbe2984" />

# Difference between CloudFromation Vs Terraform

| **Feature** | **CloudFormation** | **Terraform** | 
|------------|------------------|-------------|
| **Developer** | Amazon Web Services | HashiCorp | 
| **Cloud Support** | AWS only | Multi-cloud (AWS, Azure, GCP, etc.) |
| **Language** | JSON / YAML | HCL (HashiCorp Configuration Language) |
| **Multi-Cloud Capability** | ❌ Not supported |  ✅ Strong support |
| **State Management** | Managed by AWS (no local file needed) | Uses `.tfstate` file (local/remote) |
| **Example** | Create EC2, S3 inside AWS only | Create AWS EC2 + Azure VM + GCP storage together |
