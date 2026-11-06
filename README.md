# Learn Terraform: auto-scaling web server on AWS

A small Terraform project that deploys a highly available "Hello, World" web server on AWS: an Auto Scaling Group of EC2 instances behind an Application Load Balancer, in the default VPC. It was written to learn Terraform and to share with teammates so we could practise on the same code together.

## What it builds

```text
Internet ──► Application Load Balancer (port 80)
                 │  listener rule /* ──► target group (health check on /)
                 ▼
          Auto Scaling Group (2 to 10 × t3.micro, default VPC subnets)
                 │  each instance runs busybox httpd on port 8080
                 ▼
          "Hello, World"
```

| Resource | Purpose |
|---|---|
| `aws_launch_configuration.example` | t3.micro instance template; user data starts a web server on `var.server_port` |
| `aws_autoscaling_group.example` | Keeps 2 to 10 instances running, replaced on ELB health-check failure |
| `aws_lb.example` + `aws_lb_listener.http` | Public ALB on port 80, 404 by default |
| `aws_lb_listener_rule.asg` + `aws_lb_target_group.asg` | Forwards all paths to the instances |
| `aws_security_group.instance` / `.alb` | Allow 8080 to instances and 80 to the ALB |

## Tech stack

Terraform ≥ 1.5 · AWS provider ~> 5.100 · Amazon EC2, Auto Scaling, Elastic Load Balancing (ALB), VPC

## Getting started

### Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) 1.5 or newer
- An AWS account and credentials configured (`aws configure` or environment variables)
- A default VPC in `us-east-2` (the region set in `main.tf`)

### Deploy

```bash
git clone https://github.com/bikash9cmd/learn-terraform-project.git
cd learn-terraform-project

terraform init
terraform plan
terraform apply
```

When it finishes, Terraform prints `alb_dns_name`. After a minute or two for the instances to pass health checks:

```bash
curl http://$(terraform output -raw alb_dns_name)
# Hello, World
```

### Clean up

This creates billable resources (an ALB and at least two EC2 instances), so destroy them when you're done:

```bash
terraform destroy
```

## Variables

| Name | Default | Description |
|---|---|---|
| `server_port` | `8080` | Port the web server listens on |

## Working on it together

Pull the latest `master`, make your change on a branch, run `terraform fmt` and `terraform validate`, and open a pull request. Don't commit `.tfstate` or `.tfvars` files; they are already in `.gitignore`.

## License

Released under the [MIT License](LICENSE).
