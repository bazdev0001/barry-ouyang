# AWS EKS Terraform Module

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)
![EKS](https://img.shields.io/badge/EKS-FF9900?style=flat&logo=amazon-eks&logoColor=white)

After manually clicking through the AWS console for 3 years, I got tired of the drift. Every cluster was a snowflake. Security groups applied inconsistently. IAM policies that "worked" but nobody could explain why. Node groups with different AMIs. This Terraform module is what I wish existed when I started — opinionated where it matters, but with proper escape hatches for the edge cases.

## Why I Built This

I joined a company that had five EKS clusters. No two were alike. Standing up a new one took 2-3 days. The ops runbook was a Google Doc with screenshots. When we needed to rebuild the staging cluster after a botched upgrade, it took 11 hours and we still weren't sure we'd replicated everything correctly.

I spent six weeks extracting patterns from all five clusters, identifying the subset that should be universal defaults, and building this module. Now a new cluster is 35 minutes from `terraform apply` to running workloads.

## What's Included

### VPC and Networking
- Public/private subnet topology (workloads on private subnets, NAT gateway for egress)
- VPC flow logs to CloudWatch
- Separate subnets for pods (custom networking) to avoid IP exhaustion
- Security group rules following least-privilege — no 0.0.0.0/0 ingress

### EKS Cluster
- Managed control plane with private API endpoint option
- Envelope encryption for secrets via AWS KMS
- OIDC provider for IRSA (IAM Roles for Service Accounts) — no more node-level IAM permissions
- Cluster logging to CloudWatch (api, audit, authenticator, controllerManager, scheduler)

### Node Groups
- Managed node groups with launch template customization
- Bottlerocket AMI by default (smaller attack surface than Amazon Linux 2)
- Multiple instance type support for Spot optimization
- Automatic AMI updates via SSM Parameter Store

### Add-ons
- AWS Load Balancer Controller (replaces the old in-tree ingress)
- EBS CSI Driver with encryption by default
- VPC CNI with custom networking enabled
- CoreDNS with autoscaling configuration
- Karpenter configuration for node provisioning

### IAM and Security
- IRSA roles for all standard add-ons
- Pod security admission configuration
- AWS Secrets Manager integration template

## Usage

```hcl
module "eks" {
  source = "./modules/eks"

  cluster_name    = "production"
  cluster_version = "1.28"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids

  node_groups = {
    general = {
      instance_types = ["m5.xlarge", "m5a.xlarge", "m4.xlarge"]
      min_size       = 2
      max_size       = 20
      desired_size   = 4
    }
    compute = {
      instance_types = ["c5.2xlarge", "c5a.2xlarge"]
      min_size       = 0
      max_size       = 50
      desired_size   = 0
      capacity_type  = "SPOT"
    }
  }

  tags = {
    Environment = "production"
    Team        = "platform"
  }
}
```

## Outcomes

- New cluster provisioning: 3 days reduced to 35 minutes
- 100% infrastructure state tracked in Terraform (zero console drift)
- Module reused across 6 production clusters
- DR test: full cluster rebuild completed in 42 minutes
- IAM permissions reduced by 60% via IRSA (removing node-level permissions)

## Module Structure

```
.
├── modules/
│   ├── eks/           # Core EKS cluster
│   ├── vpc/           # VPC with EKS-optimized networking
│   ├── node-groups/   # Managed node group patterns
│   └── addons/        # Cluster add-ons (LBC, EBS CSI, etc.)
├── examples/
│   ├── basic/         # Minimal viable cluster
│   ├── production/    # Full production configuration
│   └── multi-region/  # Cross-region federation
└── test/
    └── terraform_test.go  # Terratest integration tests
```

## Prerequisites

- Terraform 1.6+
- AWS CLI configured with appropriate permissions
- kubectl

## Input Variables

| Variable | Description | Default |
|---|---|---|
| `cluster_name` | Name of the EKS cluster | required |
| `cluster_version` | Kubernetes version | `"1.28"` |
| `vpc_id` | VPC to deploy into | required |
| `subnet_ids` | Private subnets for nodes | required |
| `node_groups` | Node group configurations | `{}` |
| `enable_karpenter` | Enable Karpenter autoscaler | `true` |
| `private_api_endpoint` | Restrict API server access | `false` |

## Design Philosophy

I've made strong defaults here because consistency matters more than flexibility in most cases. If you need to deviate from the defaults, every variable has an escape hatch. But I've found that 90% of the time, the defaults are what you want in production.

The one thing I won't compromise on: IRSA over node-level IAM. I've seen too many incidents where a compromised container escalated to full EC2 instance permissions. With IRSA, the blast radius is per-pod, per-role.
# Karpenter nodepool tuning for GPU workloads
