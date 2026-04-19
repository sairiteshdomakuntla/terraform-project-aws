# Terraform AWS Web Infrastructure Project

This project provisions a complete, basic web infrastructure on AWS using Terraform.

It creates:
- A custom VPC
- Two public subnets in different Availability Zones
- An Internet Gateway and route table for internet access
- A security group for HTTP (80) and SSH (22)
- Two EC2 instances (Apache web servers) with startup scripts
- An Application Load Balancer (ALB)
- A target group and listener to route HTTP traffic to both instances
- An S3 bucket

After deployment, Terraform outputs the ALB DNS name so you can open the app in your browser.

## Architecture Summary

Traffic flow:
1. User sends HTTP request to ALB.
2. ALB forwards request to target group.
3. Target group distributes traffic across two EC2 instances in separate subnets/AZs.
4. Each EC2 instance serves a simple Apache-hosted HTML page generated from user data.

## Project Files

- `provider.tf`: Terraform and AWS provider configuration (`us-east-1`, AWS provider `6.41.0`).
- `variables.tf`: Input variable definitions (currently `cidr`).
- `main.tf`: Core AWS infrastructure resources and output.
- `userdata.sh`: Bootstrap script for web server 1.
- `userdata1.sh`: Bootstrap script for web server 2.

## Prerequisites

- Terraform installed (recommended v1.5+)
- AWS account and credentials configured locally (via AWS CLI profile, env vars, or IAM role)
- Permissions to create VPC, EC2, ALB, Security Group, and S3 resources

## Deploy

Run the following from the project root:

```bash
terraform init
terraform plan
terraform apply
```

Approve when prompted.

## Access the Application

After apply completes, Terraform prints:
- `loadbalancerdns`

Open:

```text
http://<loadbalancerdns>
```

You should see a simple HTML page from one of the backend servers.

## Destroy Infrastructure

To remove everything created by this project:

```bash
terraform destroy
```

## Current Configuration Details

- Region: `us-east-1`
- VPC CIDR: defaults to `10.0.0.0/16`
- Subnets:
	- `10.0.0.0/24` in `us-east-1a`
	- `10.0.1.0/24` in `us-east-1b`
- EC2 instances:
	- AMI: `ami-0ec10929233384c7f`
	- Type: `t2.micro`
- ALB: internet-facing, HTTP listener on port 80
- Security group ingress:
	- TCP 80 from `0.0.0.0/0`
	- TCP 22 from `0.0.0.0/0`

## Notes and Caveats

- S3 bucket names are globally unique. The current bucket name (`riteshterraformproject`) may fail if already taken.
- SSH is currently open to the world (`0.0.0.0/0`). For production, restrict SSH ingress to your IP/CIDR.
- No HTTPS listener/certificate is configured yet.
- State is currently local (`terraform.tfstate` in the project directory). For team usage, use a remote backend (for example S3 + DynamoDB lock table).

## Suggested Improvements

- Parameterize hardcoded values (AMI ID, instance type, subnet CIDRs, AZs, bucket name).
- Add tags to all resources for cost tracking and ownership.
- Add outputs for instance IDs and ALB ARN.
- Add HTTPS (ACM certificate + port 443 listener).
- Move state to remote backend for collaboration and safer state management.
