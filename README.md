# terraform-aws-alb-module

Creates an internet-facing Application Load Balancer, a single target
group (`target_type = "ip"`, required for Fargate), an HTTP listener, and
an optional HTTPS listener.

## Behavior

- If `certificate_arn` is empty → HTTP listener forwards directly to the
  target group (HTTP-only, suitable for dev).
- If `certificate_arn` is set → HTTP listener redirects (301) to HTTPS, and
  an HTTPS listener (TLS 1.3 policy) forwards to the target group.
- `enable_deletion_protection` is automatically `true` when
  `environment == "prod"`.

## Usage

```hcl
module "alb" {
  source = "git::https://github.com/<YOUR_GITHUB_ORG>/terraform-aws-alb-module.git?ref=main"

  project_name      = "hotel-bookings"
  environment       = "dev"
  common_tags       = { Project = "hotel-bookings", Environment = "dev" }
  vpc_id            = module.vpc.vpc_id
  public_subnet_ids = module.vpc.public_subnet_ids
  alb_sg_id         = module.sg[0].sg_id
  container_port    = 80
  certificate_arn   = ""   # or a real ACM ARN for HTTPS
}
```

## Inputs

| Name                    | Type       | Default | Description                                        |
|---------------------------|------------|---------|-------------------------------------------------------|
| `project_name`               | string      | –       | Used in resource naming and tags.                        |
| `environment`                  | string      | –       | e.g. `dev`, `prod`. Drives deletion protection.              |
| `common_tags`                    | map(any)     | `{}`    | Tags applied to every resource.                                |
| `vpc_id`                           | string      | –       | VPC the ALB/target group are created in.                          |
| `public_subnet_ids`                   | list(any)     | –       | Subnets the ALB is placed in (must be public).                        |
| `alb_sg_id`                              | string      | –       | Security group attached to the ALB.                                      |
| `container_port`                           | number      | `8080`  | Port the target group forwards to.                                        |
| `certificate_arn`                             | string      | `""`    | ACM cert ARN. Empty = HTTP-only.                                             |
| `alb_tags`                                       | map(any)     | `{}`    | Extra tags merged onto the ALB and target group.                                |

## Outputs

| Name                | Description                     |
|-----------------------|------------------------------------|
| `alb_dns_name`           | Public DNS name of the ALB.           |
| `alb_arn`                  | ARN of the ALB.                          |
| `target_group_arn`           | ARN of the target group (pass to ECS).       |
