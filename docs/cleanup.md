Follow these steps in order to safely remove resources and avoid dependency or billing issues.

---

## ⚠ Important
- Delete the **NAT Gateway** early to stop hourly charges.
- Terminate EC2 instances before removing subnets.
- If you created VPC Interface Endpoints (SSM etc.), delete them before route tables.

---

## 1 — Terminate EC2 instances
- EC2 → Instances → select `app-instance`, `db-instance` → Instance state → Terminate
- Wait for instance state to become *terminated*.

---

## 2 — Delete NAT Gateway & Release EIP
- VPC → NAT Gateways → select NAT → Delete
- VPC → Elastic IPs → release the EIP you allocated

---

## 3 — Delete VPC Endpoints (if created)
- VPC → Endpoints → select SSM / SSMMessages / EC2Messages / S3 → Actions → Delete

---

## 4 — Delete Security Groups
- EC2 → Network & Security → Security Groups → delete `public-sg`, `app-sg`, `db-sg`

---

## 5 — Delete Route Tables (remove associations first)
- VPC → Route Tables → for `rtb-private-app`, `rtb-private-db`, `rtb-public`:
  - Remove subnet associations
  - Delete route table

---

## 6 — Detach & Delete Internet Gateway
- VPC → Internet Gateways → detach IGW from `projectB-vpc`
- Delete IGW

---

## 7 — Delete Subnets
- VPC → Subnets → delete public / private app / private db subnets

---

## 8 — Delete the VPC
- VPC → Your VPCs → select `projectB-vpc` → Delete

---

## 9 — Delete IAM Role (optional)
- IAM → Roles → delete `EC2-SSM-Role` if not needed elsewhere

---

## 10 — Billing check
- Billing → Cost Explorer → check for any residual Cloud/Network charges
- Ensure NAT / EIP / EC2 show zero usage after cleanup

---

# Done
Your AWS account should be clean of artifacts from Project after these steps.
