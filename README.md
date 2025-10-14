# AWS High Availability & Monitoring Infrastructure

* This project demonstrates a **production-ready, highly available architecture on AWS** with automated monitoring using CloudWatch. It provisions scalable EC2 instances behind an Application Load Balancer (ALB), configures Auto Scaling Groups (ASG), and implements metrics-based alerts via CloudWatch and SNS.
* A key feature of this setup is that the AMI used for launching EC2 instances in the Auto Scaling Group is created from an existing main instance, ensuring that the application and configuration are consistent across all new instances.
---

## 🔹 Project Highlights

* **High Availability:**

  * Auto Scaling Group with minimum, desired, and maximum instance counts
  * Instances deployed across multiple subnets (private/public) for fault tolerance
  * Load balancing using Application Load Balancer (HTTP/HTTPS)

* **AMI from Existing Instance**

* Launches EC2 instances from a pre-configured AMI created from a main existing instance.
* Ensures all instances have consistent configuration and installed software.

* **Monitoring & Alerts:**

  * CloudWatch Agent installed on EC2 instances
  * Metrics collected: CPU (Idle, System, User), Memory %, Disk usage %
  * CloudWatch alarms trigger SNS email notifications if thresholds are exceeded

* **Secure Infrastructure:**

  * IAM roles and instance profiles with least privilege for EC2
  * S3 bucket and DynamoDB table backend for Terraform state management
  * Secure networking using VPC, public/private subnets, NAT Gateway, and route tables

* **Infrastructure as Code:**

  * Full deployment automated using Terraform
  * Launch template configures EC2 instances and CloudWatch agent via `user_data.sh`
  * Terraform backend supports versioning and state locking

---

## 🔹 Architecture Overview

```
              +-----------------+
              |  Application    |
              | Load Balancer   |
              +--------+--------+
                       |
           +-----------+-----------+
           |                       |
   EC2 Instance 1           EC2 Instance 2
   (Private Subnet)         (Private Subnet)
           |                       |
     CloudWatch Agent monitors CPU, Memory, Disk
           |
       CloudWatch Metrics & Alarms
           |
        SNS Notifications (Email)
```

---

## 🔹 Tech Stack

| Technology   | Purpose                                    |
| ------------ | ------------------------------------------ |
| Terraform    | Infrastructure provisioning and management |
| AWS EC2      | Compute resources                          |
| AWS VPC      | Networking                                 |
| ALB          | Load balancing                             |
| Auto Scaling | High availability & dynamic scaling        |
| CloudWatch   | Metrics collection & alerting              |
| SNS          | Notification delivery (email)              |
| IAM Roles    | Secure instance permissions                |

---

## 🔹 Getting Started

### Prerequisites

* AWS CLI configured with proper credentials
* Terraform >= 1.8.5
* Existing EC2 instance to create AMI from.
* GitHub Actions for CI/CD automation (optional)
* SSH key for EC2 access

GitHub Secrets Required

To run the workflow (.github/workflows/terraform.yml), set the following secrets in your GitHub repository:
| Secret Name             | Purpose                                |
| ----------------------- | -------------------------------------- |
| `AWS_ACCESS_KEY`        | AWS Access Key for Terraform & AWS CLI |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Key for Terraform & AWS CLI |
| `KEY_NAME`              | SSH Key name for EC2 instance access   |

### 1️⃣ Terraform Backend Setup

Run `create-backend.sh` to create:

* S3 bucket for Terraform state storage
* DynamoDB table for state locking

```bash
./create-backend.sh
```

### 2️⃣ Deploy Infrastructure

Via GitHub Actions workflow (`.github/workflows/terraform.yml`) or manually:

```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

### 3️⃣ CloudWatch Monitoring

* CloudWatch agent installed via EC2 launch template (`user_data.sh`)
* Collects CPU, memory, and disk metrics every minute
* SNS notifications sent for high CPU utilization (>80%)

---

## 🔹 Terraform Variables (`terraform.tfvars`)

```hcl
aws_region = "us-east-1"
project = "app"
source_instance_id = "i-xxxxxxxxxxxx"
vpc_cidr = "10.10.0.0/16"
public_subnet_cidrs = ["10.10.1.0/24", "10.10.2.0/24"]
private_subnet_cidrs = ["10.10.101.0/24", "10.10.102.0/24"]
instance_type = "t2.micro"
min_size = 1
desired_capacity = 2
max_size = 5
alert_email = "your_email@example.com"
acm_cert_arn = "" # optional HTTPS
```

---

## 🔹 Outputs

* ALB DNS name
* ASG name
* AMI ID
* VPC ID

---

