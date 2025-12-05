# How to Build a Secure 3-Tier VPC (Project B) — Step-by-Step

This runbook documents a console-first deployment of a 3-tier VPC:
- Public subnets (NAT / ALB / Bastion)
- Private App subnets (outbound via NAT)
- Private DB subnets (no internet)
- SSM Session Manager access (no SSH keys)
- Security groups, route tables, IGW, NAT

**Region used in examples:** `us-east-1` (N.Virginia) — adapt if you choose another region.

---

## Project CIDR plan 
- VPC: `10.0.0.0/16`
- public-az1: `10.0.1.0/24` (us-east-1a)
- public-az2: `10.0.2.0/24` (us-east-1b)
- private-app-az1: `10.0.10.0/24`
- private-app-az2: `10.0.11.0/24`
- private-db-az1: `10.0.20.0/24`
- private-db-az2: `10.0.21.0/24`

---

# PHASE 1 — VPC & Subnets

### Step 1 — Create the VPC
1. Console → VPC → **Your VPCs** → **Create VPC**
2. Type: **VPC only**
3. Name: `projectB-vpc`  
   IPv4 CIDR: `10.0.0.0/16`  
   Tenancy: Default
4. Create.

**WHY:** Foundation of the network — isolates resources from other AWS accounts.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/1010c2d7ea6f63a633570b2ecb31420b1bf86004/images/vpc-create.png)

---

### Step 2 — Create Public Subnets (AZ explicit)
1. VPC → Subnets → Create Subnet
   - VPC: `projectB-vpc`
   - Name: `public-az1`
   - AZ: `us-east-1a`
   - CIDR: `10.0.1.0/24`
2. Repeat for `public-az2` in `us-east-1b` CIDR `10.0.2.0/24`.

**WHY:** Hosts NAT gateway and any public-facing endpoints.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/b6224fe9c1d1bb91f67497385192e29c067ebaf6/images/subnets-public.png)

---

### Step 3 — Create Private App Subnets (AZ explicit)
1. Subnet: `private-app-az1` → AZ `us-east-1a` → `10.0.10.0/24`
2. Subnet: `private-app-az2` → AZ `us-east-1b` → `10.0.11.0/24`

**WHY:** App servers live here and access internet via NAT only.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/subnets-private-app.png)

---

### Step 4 — Create Private DB Subnets (AZ explicit)
1. Subnet: `private-db-az1` → AZ `us-east-1a` → `10.0.20.0/24`
2. Subnet: `private-db-az2` → AZ `us-east-1b` → `10.0.21.0/24`

**WHY:** DBs must be fully isolated; no NAT route by default.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/subnets-db.png)

---

# PHASE 2 — IGW + Public Route Table

### Step 5 — Create Internet Gateway (IGW) & Attach
1. VPC → Internet Gateways → Create → `projectB-igw`
2. Select IGW → Actions → Attach to VPC → choose `projectB-vpc`

**WHY:** Allows public subnet resources to reach the Internet.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/igw-create.png)

---

### Step 6 — Create Public Route Table and Associate
1. VPC → Route Tables → Create → Name: `rtb-public` (VPC = `projectB-vpc`)
2. Edit Routes → Add: `0.0.0.0/0` → Target: `projectB-igw`
3. Subnet Associations → add: `public-az1`, `public-az2`

**WHY:** Exposes public subnets to internet via IGW.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/route-public.png)

---

# PHASE 3 — NAT + Private Route Tables

### Step 7 — Allocate Elastic IP for NAT
1. VPC → Elastic IPs → Allocate → Name: `projectB-nat-eip`

**WHY:** NAT requires a public IP.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/eip-allocate.png)

---

### Step 8 — Create NAT Gateway (single NAT in AZ1)
1. VPC → NAT Gateways → Create
   - Subnet: `public-az1`
   - Elastic IP: `projectB-nat-eip`
   - Name: `projectB-nat`

**WHY:** Provides outbound internet for private app subnets.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/nat-create.png)

---

### Step 9 — Private App Route Table → NAT
1. Route Tables → Create → `rtb-private-app` (VPC = `projectB-vpc`)
2. Edit Routes → Add: `0.0.0.0/0` → Target: NAT Gateway ID
3. Associate subnets: `private-app-az1`, `private-app-az2`

**WHY:** Routes app traffic outbound via NAT, keeps inbound closed.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/route-private-app.png)

---

### Step 10 — Private DB Route Table (NO internet)
1. Create route table `rtb-private-db` and **do not** add `0.0.0.0/0`
2. Associate: `private-db-az1`, `private-db-az2`

**WHY:** Keeps DB tier isolated; best practice for security.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/route-private-db.png)

---

# PHASE 4 — SECURITY GROUPS & (Optional NACLs)

### Step 11 — SG: Public (`public-sg`)
- Inbound:
  - HTTP (80) / HTTPS (443) from `0.0.0.0/0` if ALB
  - SSH (22) from *your IP only* if bastion
- Outbound: all

**WHY:** Gate entry to public endpoints.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/public-sg.png)

---

### Step 12 — SG: App (`app-sg`)
- Inbound:
  - App port (e.g., 80) from `public-sg` (security group reference)
- Outbound:
  - Allow all or restrict to DB SG as needed

**WHY:** Allow only ALB (or bastion) to talk to app servers.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/app-sg.png)

---

### Step 13 — SG: DB (`db-sg`)
- Inbound:
  - MySQL (3306) or Postgres (5432) from `app-sg` only
- Outbound:
  - None or limited

**WHY:** Ensures DB is accessible only by app servers.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/db-sg.png)

---

# PHASE 5 — LAUNCH EC2 INSTANCES (SSM)

### Step 14 — Create IAM Role for SSM
1. IAM → Roles → Create role → Trusted entity: EC2
2. Attach policy: `AmazonSSMManagedInstanceCore`
3. Name: `EC2-SSM-Role`

**WHY:** Enables Session Manager without SSH keys.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/iam-role-ssm.png)

---

### Step 15 — Launch App EC2 (private-app-az1)
1. EC2 → Launch Instance
   - AMI: Amazon Linux 2023
   - Type: `t3.micro`
   - VPC: `projectB-vpc`
   - Subnet: `private-app-az1` (no public IP)
   - IAM Role: `EC2-SSM-Role`
   - Security Group: `app-sg`
   - Skip key pair (optional)
2. Launch

**WHY:** Validates private connectivity and SSM works (via NAT).

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/ec2-app-launch.png)

---

### Step 16 — Launch DB EC2 (private-db-az1)
Repeat as above:
- Subnet: `private-db-az1`
- Security Group: `db-sg`
- IAM Role: optional (see design decision)

**Design note:** DB usually has no SSM if you want full isolation. See README.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/ec2-db-launch.png)

---

# PHASE 6 — VALIDATION & TESTS

### Test A — SSM connection to app instance
1. Console → Systems Manager → Session Manager → Start session
2. Select `app-instance`

Expected: interactive shell

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/ssm-session.png)

---

### Test B — Outbound internet via NAT (from app)
```bash
curl -I https://amazon.com

```
Expected: HTTP response or 301 redirect.

![](https://github.com/vikasreddy98/Secure-3-Tier-VPC-on-AWS/blob/9c0e8e6d442d61b931e6ab65286b0f1b285ee4b1/images/nat-test.png)

#End of HowTo#
