# Terraform Jenkins Infrastructure

A production-ready Infrastructure as Code (IaC) solution for deploying Jenkins on AWS using Terraform. This project automates the provisioning of a complete AWS infrastructure with networking, security, load balancing, and SSL/TLS certificate management.

**Created by:** Harmeet Singh

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Getting Started](#getting-started)
- [AWS Resources](#aws-resources)
- [Outputs](#outputs)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

This Terraform project automates the deployment of Jenkins CI/CD server on AWS with the following features:

- **Automated Infrastructure Provisioning**: VPC, subnets, security groups, and routing
- **Jenkins Installation**: Automated Jenkins and Java installation on EC2
- **Load Balancing**: Application Load Balancer (ALB) for traffic distribution
- **SSL/TLS Security**: AWS Certificate Manager (ACM) integration with HTTPS
- **Domain Management**: Route 53 DNS configuration with custom domain support
- **High Availability**: Multi-AZ support with public and private subnets
- **Infrastructure as Code**: Modular, reusable Terraform configuration

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         AWS Account                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    VPC (11.0.0.0/16)                      │   │
│  │                                                             │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │         Public Subnets (Multi-AZ)                  │ │   │
│  │  │  • eu-west-1a: 11.0.1.0/24                         │ │   │
│  │  │  • eu-west-1b: 11.0.2.0/24                         │ │   │
│  │  │                                                     │ │   │
│  │  │  ┌────────────────────────────────────────────┐   │ │   │
│  │  │  │     Application Load Balancer (ALB)        │   │ │   │
│  │  │  │  • Port 80 (HTTP)  → Port 8080             │   │ │   │
│  │  │  │  • Port 443 (HTTPS) → Port 8080 with SSL   │   │ │   │
│  │  │  └────────────────────────────────────────────┘   │ │   │
│  │  │                     ↓                               │ │   │
│  │  │  ┌────────────────────────────────────────────┐   │ │   │
│  │  │  │      EC2 Instance (t2.medium)              │   │ │   │
│  │  │  │  • Jenkins on Port 8080                    │   │ │   │
│  │  │  │  • Ubuntu 20.04 LTS                        │   │ │   │
│  │  │  │  • Java 11 + Jenkins + Terraform           │   │ │   │
│  │  │  └────────────────────────────────────────────┘   │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  │                                                             │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │         Private Subnets (Multi-AZ)                 │ │   │
│  │  │  • eu-west-1a: 11.0.3.0/24 (Reserved for future)  │ │   │
│  │  │  • eu-west-1b: 11.0.4.0/24 (Reserved for future)  │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  │                                                             │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │           Internet Gateway (IGW)                    │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────┘ │   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           Route 53 (DNS Management)                       │   │
│  │  • Domain: jenkins.jhooq.org                             │   │
│  │  • Alias Record → ALB DNS Name                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │     AWS Certificate Manager (ACM)                         │   │
│  │  • SSL/TLS Certificate for jenkins.jhooq.org            │   │
│  │  • DNS Validation via Route 53                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Prerequisites

### Required Tools

- **Terraform** >= 1.0
  ```bash
  terraform version
  ```

- **AWS CLI** >= 2.0
  ```bash
  aws --version
  ```

- **AWS Account** with appropriate permissions
- **Valid AWS Credentials** configured locally

### AWS Permissions

Your AWS user/role must have permissions for:
- VPC and Subnet management
- EC2 instance creation
- Security Group configuration
- Load Balancer (ALB) management
- Route 53 DNS management
- ACM Certificate management
- S3 (for remote state)

### Local Setup

1. **Install Terraform**:
   ```bash
   # macOS
   brew install terraform

   # Linux/Ubuntu
   sudo apt-get update && sudo apt-get install terraform

   # Windows
   choco install terraform
   ```

2. **Configure AWS Credentials**:
   ```bash
   aws configure
   # Enter your AWS Access Key ID
   # Enter your AWS Secret Access Key
   # Enter default region (e.g., eu-west-1)
   # Enter default output format (json)
   ```

3. **Generate SSH Key Pair** (if not exists):
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws_ec2_terraform -N ""
   ```

## Project Structure

```
terraform-jenkins/
├── README.md                           # Project documentation
├── main.tf                             # Root module configuration
├── variables.tf                        # Variable definitions
├── outputs.tf                          # Output definitions
├── terraform.tfvars                    # Variable values
├── provider.tf                         # AWS provider configuration
├── remote_backend_s3.tf               # Remote state backend (S3)
│
├── networking/                         # VPC & Networking Module
│   └── main.tf                        # VPC, Subnets, IGW, Route Tables
│
├── security-groups/                    # Security Groups Module
│   └── main.tf                        # Security group rules
│
├── jenkins/                            # Jenkins EC2 Module
│   └── main.tf                        # EC2 instance configuration
│
├── load-balancer/                      # ALB Module
│   └── main.tf                        # Application Load Balancer
│
├── load-balancer-target-group/         # Target Group Module
│   └── main.tf                        # ALB Target Group
│
├── hosted-zone/                        # Route 53 Module
│   └── main.tf                        # DNS configuration
│
├── certificate-manager/                # ACM Module
│   └── main.tf                        # SSL/TLS certificates
│
└── jenkins-runner-script/              # Installation Scripts
    └── jenkins-installer.sh           # Jenkins & Terraform setup script
```

## Configuration

### Key Variables (terraform.tfvars)

```hcl
bucket_name = "dev-proj-1-jenkins-remote-state-bucket-123456"

# Networking
vpc_cidr             = "11.0.0.0/16"
vpc_name             = "dev-proj-jenkins-eu-west-vpc-1"
cidr_public_subnet   = ["11.0.1.0/24", "11.0.2.0/24"]
cidr_private_subnet  = ["11.0.3.0/24", "11.0.4.0/24"]
eu_availability_zone = ["eu-west-1a", "eu-west-1b"]

# EC2 Configuration
public_key = "ssh-rsa AAAAB3NzaC1yc2EA..."  # Your public SSH key
ec2_ami_id = "ami-0694d931cee176e7d"        # Ubuntu 20.04 AMI
```

### Customization Guide

#### Change AWS Region

Edit `provider.tf`:
```hcl
provider "aws" {
  region = "us-east-1"  # Change from eu-west-1 to desired region
}
```

Update availability zones in `terraform.tfvars`:
```hcl
eu_availability_zone = ["us-east-1a", "us-east-1b"]
```

#### Change Jenkins Domain

Edit `terraform.tfvars` and the references in `main.tf`:
```hcl
# In main.tf, update:
domain_name = "your-domain.com"
```

#### Change Instance Type

Edit `main.tf`:
```hcl
instance_type = "t2.large"  # Change from t2.medium
```

#### Adjust Networking CIDR Blocks

Edit `terraform.tfvars`:
```hcl
vpc_cidr             = "10.0.0.0/16"  # Change VPC CIDR
cidr_public_subnet   = ["10.0.1.0/24", "10.0.2.0/24"]
cidr_private_subnet  = ["10.0.3.0/24", "10.0.4.0/24"]
```

## Getting Started

### Step 1: Clone and Navigate

```bash
cd terraform-jenkins
```

### Step 2: Initialize Terraform

```bash
terraform init
```

This will:
- Download required providers (AWS)
- Initialize the local Terraform working directory
- Configure the remote backend (if defined)

### Step 3: Validate Configuration

```bash
terraform validate
```

### Step 4: Plan Infrastructure

```bash
terraform plan -out=tfplan
```

Review the planned changes and output. This shows:
- Resources to be created
- Changes to existing resources
- Destruction of removed resources

### Step 5: Apply Configuration

```bash
terraform apply tfplan
```

Or directly without saving plan:
```bash
terraform apply
```

**Expected Time**: 5-10 minutes for complete infrastructure provisioning.

### Step 6: Access Jenkins

After successful deployment:

1. **Get ALB DNS Name**:
   ```bash
   terraform output aws_lb_dns_name
   ```

2. **Get EC2 Public IP**:
   ```bash
   terraform output dev_proj_1_ec2_instance_public_ip
   ```

3. **Access Jenkins**:
   - Via Domain: `https://jenkins.jhooq.org` (once DNS propagates)
   - Via ALB DNS: `http://<alb-dns-name>`
   - Via EC2 Direct: `http://<ec2-public-ip>:8080`

4. **SSH into Jenkins Server**:
   ```bash
   ssh -i ~/.ssh/aws_ec2_terraform ubuntu@<ec2-public-ip>
   ```

5. **Retrieve Jenkins Admin Password**:
   ```bash
   ssh -i ~/.ssh/aws_ec2_terraform ubuntu@<ec2-public-ip> \
       sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

## AWS Resources

### Networking Resources

| Resource | Details |
|----------|----------|
| **VPC** | 11.0.0.0/16 CIDR block |
| **Public Subnets** | 2 subnets (11.0.1.0/24, 11.0.2.0/24) in different AZs |
| **Private Subnets** | 2 subnets (11.0.3.0/24, 11.0.4.0/24) in different AZs |
| **Internet Gateway** | Provides internet access for public subnets |
| **Route Tables** | Separate routing for public and private subnets |

### Compute Resources

| Resource | Details |
|----------|----------|
| **EC2 Instance** | t2.medium, Ubuntu 20.04 LTS |
| **Key Pair** | For SSH access |
| **Software** | Java 11, Jenkins, Terraform 1.6.5 |

### Load Balancing Resources

| Resource | Details |
|----------|----------|
| **ALB** | Application Load Balancer in public subnets |
| **Listeners** | Port 80 (HTTP) and 443 (HTTPS) |
| **Target Group** | Routes traffic to Jenkins on port 8080 |
| **Health Checks** | Path: /login, Interval: 5s, Timeout: 2s |

### Security Resources

| Resource | Details |
|----------|----------|
| **SG 1** | Allows SSH (22), HTTP (80), HTTPS (443) |
| **SG 2** | Allows Jenkins port 8080 |

### DNS & Certificate Resources

| Resource | Details |
|----------|----------|
| **Route 53** | DNS management for jenkins.jhooq.org |
| **ACM Certificate** | SSL/TLS certificate with DNS validation |

## Outputs

After successful deployment, retrieve outputs:

```bash
# Get ALB DNS Name
terraform output aws_lb_dns_name

# Get Jenkins EC2 Instance ID
terraform output jenkins_ec2_instance_ip

# Get Jenkins EC2 Public IP
terraform output dev_proj_1_ec2_instance_public_ip

# Get all outputs
terraform output
```

### Output Variables

- `aws_lb_dns_name`: Application Load Balancer DNS name
- `aws_lb_zone_id`: ALB Hosted Zone ID
- `jenkins_ec2_instance_ip`: Jenkins EC2 instance ID
- `dev_proj_1_ec2_instance_public_ip`: Jenkins EC2 public IP address
- `ssh_connection_string_for_ec2`: SSH command to connect to Jenkins server

## Security

### Best Practices Implemented

1. **Network Isolation**:
   - Public subnets for ALB and bastion access
   - Private subnets for future internal resources
   - Strict security group rules

2. **Access Control**:
   - SSH access restricted (currently open to 0.0.0.0/0 - should be restricted to your IP)
   - Jenkins accessible via ALB with SSL/TLS
   - EC2 metadata protected with IMDSv2

3. **Encryption**:
   - SSL/TLS certificates via AWS ACM
   - HTTPS enforced on ALB listeners

### Security Recommendations

⚠️ **Before Production Deployment**:

1. **Restrict SSH Access**:
   ```hcl
   # Edit security-groups/main.tf
   ingress {
     description = "Allow remote SSH from specific IP"
     cidr_blocks = ["YOUR.IP.ADDRESS/32"]  # Change to your IP
     from_port   = 22
     to_port     = 22
     protocol    = "tcp"
   }
   ```

2. **Enable MFA** for AWS console access

3. **Use AWS Secrets Manager** for Jenkins credentials

4. **Enable CloudTrail** for audit logging

5. **Configure CloudWatch** for monitoring and alerts

6. **Setup backup** for Jenkins home directory

## Troubleshooting

### Issue: Terraform initialization fails

**Solution**:
```bash
rm -rf .terraform terraform.lock.hcl
terraform init
```

### Issue: AWS credentials not found

**Solution**:
```bash
aws configure
# Or set environment variables
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"
export AWS_DEFAULT_REGION="eu-west-1"
```

### Issue: Jenkins not accessible after deployment

**Steps to debug**:
```bash
# 1. Check EC2 instance status
aws ec2 describe-instances --region eu-west-1

# 2. Check ALB target health
aws elbv2 describe-target-health --target-group-arn <arn> --region eu-west-1

# 3. SSH into instance
ssh -i ~/.ssh/aws_ec2_terraform ubuntu@<ec2-ip>

# 4. Check Jenkins service
sudo systemctl status jenkins

# 5. Check logs
sudo tail -f /var/log/jenkins/jenkins.log
```

### Issue: DNS resolution takes too long

**Solution**: DNS propagation can take 15-30 minutes. Meanwhile, use the ALB DNS name or EC2 public IP directly.

### Issue: SSL certificate not validating

**Steps**:
```bash
# 1. Check ACM certificate status
aws acm describe-certificate --certificate-arn <arn> --region eu-west-1

# 2. Verify Route 53 records
aws route53 list-resource-record-sets --hosted-zone-id <zone-id>

# 3. Check validation records manually
dig jenkins.jhooq.org
```

### Issue: ALB returning 502 Bad Gateway

**Causes and Solutions**:
- Jenkins not running: SSH and check `sudo systemctl status jenkins`
- Health check path incorrect: Verify `/login` returns 200
- Security groups not allowing traffic: Check SG rules
- Instance not in target group: Check ALB target health

### Issue: Terraform state conflicts

**Solution**:
```bash
# Refresh state
terraform refresh

# View state
terraform state list

# Remove resource from state (if needed)
terraform state rm 'resource.name'
```

## Cleanup

### Destroy All Resources

```bash
terraform destroy
```

Review the plan carefully before confirming. This will remove:
- EC2 instances
- Load balancers
- VPC and all networking resources
- Security groups
- SSL certificates
- Route 53 records

### Partial Cleanup

```bash
# Remove specific resource
terraform destroy -target=module.jenkins

# Destroy with auto-approval (use carefully)
terraform destroy -auto-approve
```

## Contributing

Contributions are welcome! To contribute:

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature`
3. **Make changes** and test thoroughly
4. **Commit with descriptive messages**: `git commit -m 'Add your feature'`
5. **Push to branch**: `git push origin feature/your-feature`
6. **Create a Pull Request**

### Testing Changes

Before submitting:
```bash
terraform fmt -recursive          # Format code
terraform validate               # Validate syntax
terraform plan                   # Review changes
```

## Additional Resources

### Terraform Documentation
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Language Documentation](https://www.terraform.io/language)

### AWS Documentation
- [VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [ACM Documentation](https://docs.aws.amazon.com/acm/)

### Jenkins Documentation
- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [Jenkins Configuration as Code](https://plugins.jenkins.io/configuration-as-code/)

## Support

For issues, questions, or suggestions:
- Check existing GitHub issues
- Create a new GitHub issue with detailed information
- Include terraform output and logs

## License

This project is available under the MIT License. See LICENSE file for details.

---

**Author:** Harmeet Singh  
**Last Updated:** January 2026  
**Status:** Production Ready
