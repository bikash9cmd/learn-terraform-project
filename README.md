# Learn Terraform Project — Auto Scaling Web Cluster on AWS

A Terraform project that deploys a highly available web server cluster on AWS: an Auto Scaling Group of EC2 instances behind an Application Load Balancer, in the default VPC of `us-east-2`. Built while learning Terraform and shared so teammates can work on it together.

## What it creates

| Resource | Purpose |
| --- | --- |
| `aws_launch_template.example` | Launch template for `t3.micro` instances that serve "Hello, World" with BusyBox on port 8080 |
| `aws_autoscaling_group.example` | 2 to 10 instances across the default VPC subnets, using ELB health checks |
| `aws_lb.example` | Public Application Load Balancer |
| `aws_lb_listener.http` | HTTP listener on port 80 (returns 404 by default) |
| `aws_lb_listener_rule.asg` | Forwards all paths to the target group |
| `aws_lb_target_group.asg` | Health-checks instances on `/` |
| `aws_security_group.instance` / `.alb` | Allow 8080 to instances and 80 to the load balancer |

## Architecture

```
Internet ──HTTP :80──▶ Application Load Balancer ──:8080──▶ Auto Scaling Group (2–10 EC2, default VPC subnets)
```

## Prerequisites

- Terraform >= 1.5
- An AWS account and credentials configured (`aws configure` or environment variables)

## Usage

```bash
terraform init
terraform plan
terraform apply

# open the load balancer URL (it can take a minute or two for instances to pass health checks)
curl http://$(terraform output -raw alb_dns_name)

terraform destroy   # remove everything when finished to avoid charges
```

## Variables

| Name | Description | Default |
| --- | --- | --- |
| `server_port` | Port the web server listens on | `8080` |

## Outputs

| Name | Description |
| --- | --- |
| `alb_dns_name` | DNS name of the load balancer |

## Notes

- The AMI `ami-0fb653ca2d3203ac1` is an Ubuntu image in `us-east-2`; change it if you use another region.
- State is stored locally. For team use, move it to an S3 backend with state locking.
