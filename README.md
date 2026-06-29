# 🏛️ 3-Tier Application on AWS

> A cloud-native 3-tier web application deployed on AWS — demonstrating clean separation of presentation, application, and data layers, with all infrastructure provisioned as code via Terraform and deployed through an automated CI/CD pipeline.

Built to reflect real patterns used in production environments across financial services and e-commerce platforms.

---

## 🏗️ Architecture

```
                        ┌─────────────────────────────────────────┐
                        │              CloudFront CDN              │
                        │         (Global edge caching)            │
                        └───────────────────┬─────────────────────┘
                                            │
                        ┌───────────────────▼─────────────────────┐
                        │          Application Load Balancer        │
                        │      (SSL termination, path routing)      │
                        └──────────┬────────────────┬─────────────┘
                                   │                │
              ┌────────────────────▼──┐    ┌────────▼──────────────────┐
              │   ECS Fargate Tasks   │    │      S3 Static Assets      │
              │   (Application Layer) │    │   (Frontend / JS / CSS)    │
              │   Auto Scaling Group  │    └───────────────────────────┘
              └────────────┬──────────┘
                           │
              ┌────────────▼──────────────────────────────────────────┐
              │                    Data Layer                          │
              │   ┌─────────────────┐    ┌────────────────────────┐   │
              │   │  RDS PostgreSQL  │    │  ElastiCache (Redis)   │   │
              │   │   (Multi-AZ)    │    │    (Session cache)     │   │
              │   └─────────────────┘    └────────────────────────┘   │
              └───────────────────────────────────────────────────────┘

VPC: Public subnets (ALB) → Private subnets (App) → Isolated subnets (DB)
```

---

## ☁️ AWS Services

| Layer | Service | Purpose |
|-------|---------|---------|
| Edge | CloudFront | Global CDN, SSL offloading, WAF integration |
| Frontend | S3 | Static asset delivery |
| Load Balancing | ALB | Path-based routing, health checks, SSL termination |
| Application | ECS Fargate | Serverless containers, no EC2 management |
| Auto Scaling | Application Auto Scaling | Scale tasks on CPU/memory/request count |
| Database | RDS PostgreSQL (Multi-AZ) | Transactional data, automated failover |
| Caching | ElastiCache Redis | Session management, query caching |
| Secrets | AWS Secrets Manager | DB credentials with automated rotation |
| Networking | VPC, NAT Gateway | Public/private/isolated subnet architecture |
| Security | WAF, Security Groups, IAM | Defence in depth |
| Monitoring | CloudWatch, Container Insights | Logs, metrics, alarms |

---

## 🔒 Security Design

- **No public database access** — RDS in isolated subnet, no internet gateway route
- **Least-privilege IAM** — ECS task roles scoped to minimum required permissions
- **Secrets Manager rotation** — DB credentials rotate automatically, no hardcoded passwords
- **WAF rules** — SQL injection, XSS, rate limiting at the edge
- **VPC Flow Logs** — all traffic logged to S3 for audit
- **Encryption everywhere** — RDS encrypted at rest (KMS), S3 SSE, ALB SSL/TLS

---

## 📁 Repository Structure

```
├── terraform/
│   ├── modules/
│   │   ├── vpc/              # VPC, subnets, routing, NAT Gateway
│   │   ├── alb/              # Application Load Balancer + target groups
│   │   ├── ecs/              # ECS cluster, service, task definition
│   │   ├── rds/              # RDS PostgreSQL Multi-AZ
│   │   ├── elasticache/      # Redis cluster
│   │   ├── cloudfront/       # CloudFront distribution + S3 origin
│   │   ├── waf/              # WAF web ACL
│   │   └── iam/              # Task execution + task roles
│   └── environments/
│       ├── dev/
│       └── production/
├── app/                      # Application source code
│   ├── frontend/             # Static assets (JS, CSS, HTML)
│   └── backend/              # API service
├── docker/
│   └── Dockerfile            # Multi-stage build, non-root user, minimal image
└── .github/
    └── workflows/
        ├── build-scan.yml    # Docker build + Trivy scan
        └── deploy.yml        # Terraform plan/apply + ECS deploy
```

---

## 🚀 Deployment

```bash
# Provision infrastructure
cd terraform/environments/production
terraform init
terraform plan -out=tfplan
terraform apply tfplan

# Build and push container
docker build -t my-app:latest .
docker tag my-app:latest <account>.dkr.ecr.<region>.amazonaws.com/my-app:latest
aws ecr get-login-password | docker login --username AWS --password-stdin <ecr-url>
docker push <account>.dkr.ecr.<region>.amazonaws.com/my-app:latest

# Deploy to ECS (handled by GitHub Actions pipeline)
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
```

---

## 🛠️ Tech Stack

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=flat&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)

---

## 📖 Related

- [Mastering_AWS_Terraform](https://github.com/SaiJithendraGonji/Mastering_AWS_Terraform) — Terraform modules used here
- [ENDTOEND_DEVSECOPS](https://github.com/SaiJithendraGonji/ENDTOEND_DEVSECOPS) — Security pipeline that scans this application
- [HELM-Charts](https://github.com/SaiJithendraGonji/HELM-Charts) — Kubernetes alternative deployment for this app
