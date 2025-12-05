# Secure 3-Tier VPC on AWS (S3/CloudFront Project Companion)

**Short:** This project implements a production-style 3-tier VPC (public, private app, private DB) across 2 AZs using AWS Console only — no SSH keys required (SSM used for private access). It's designed for portfolio demonstration and SAA-C03 exam relevance.

---

## Architecture (high level)
- VPC: `10.0.0.0/16`  
- Public subnets: `10.0.1.0/24`, `10.0.2.0/24` (AZ1, AZ2)  
- Private app subnets: `10.0.10.0/24`, `10.0.11.0/24`  
- Private DB subnets: `10.0.20.0/24`, `10.0.21.0/24`  
- IGW, NAT Gateway (single NAT in AZ1), route tables, SGs for app & db  
- SSM (Session Manager) for console access to private instances

---

##  Repo structure

```
project/
├── docs/
│   ├── cleanup.md
│   └── howto.md
│
├── images/
│   ├── app-sg.png
│   ├── db-sg.png
│   ├── ec2-app-launch.png
│   ├── ec2-db-launch.png
│   ├── eip-allocate.png
│   ├── iam-role-ssm.png
│   ├── igw-create.png
│   ├── nat-create.png
│   ├── nat-test.png
│   ├── public-sg.png
│   ├── route-private-app.png
│   ├── route-private-db.png
│   ├── route-public.png
│   ├── ssm-session.png
│   ├── subnets-db.png
│   ├── subnets-private-app.png
│   ├── subnets-public.png
│   └── vpc-create.png
│
└── README.md
```
---

##  What I built (evidence)
- VPC and subnets (public/app/db) across 2 AZs.
- Internet Gateway + Public route table.
- NAT Gateway + Private route table for app subnets.
- App and DB EC2 instances (private).
- Security groups with SG-to-SG rules.
- SSM role & usage for secure instance access.

See ![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/34c2c3ec3f710b686b3c998e7753f1d5d866166f/docs/howto.md) for console steps and `![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/34c2c3ec3f710b686b3c998e7753f1d5d866166f/docs/cleanup.md)` for safe deletion sequence.

---

##  Validation checklist
-  App instance reachable via SSM (Session Manager)
-  App instance can access internet via NAT (`curl -I https://amazon.com`)
-  App → DB connectivity allowed via SG rules (`nc -zv <db-ip> 3306`)
-  DB instance has **no public IP**
-  NAT Gateway deleted after tests to avoid charges

---

##  Cost note
- Small EC2 (t3.micro) — minimal while running  
- NAT Gateway — hourly charges; delete after testing  
- SSM Interface endpoints (optional) incur small interface endpoint charges

---

## 📧 Contact
Your Name — LinkedIn — Email

