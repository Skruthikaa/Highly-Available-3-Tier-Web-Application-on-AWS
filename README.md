# Highly Available 3-Tier Web Application on AWS

---

### Overview

This document describes the process of building a **highly available, scalable, and fault-tolerant architecture** on Amazon Web Services using core services such as VPC, EC2, Load Balancer, Auto Scaling, CloudWatch, and S3.

---

##  Step 1: VPC Creation

### Theory

A **Virtual Private Cloud (VPC)** is an isolated network environment in AWS that allows control over IP addressing, routing, and security.

##  Configuration

* VPC Name: `ecommerce-vpc`
* IPv4 CIDR Block: `10.0.0.0/16`

This CIDR range provides a large pool of private IP addresses for internal resources.

---

##  Screenshots

![VPC-1](https://github.com/user-attachments/assets/9ddabf71-aa73-4159-b908-85de9f242333)
![VPC-2](https://github.com/user-attachments/assets/32968b07-a5da-4f93-9269-b74c939d069a)
![VPC-3](https://github.com/user-attachments/assets/25adf79a-2d84-4510-b78b-c2123e578a31)

---

##  Step 2: Subnet Configuration (Multi-AZ)

###  Theory

Subnets divide the VPC into smaller segments. They are categorized as:

* **Public Subnets**: Allow internet access
* **Private Subnets**: Restrict direct internet access

Deploying subnets across multiple Availability Zones improves availability and fault tolerance.

---

###  Configuration

### Public Subnets

* `public-subnet-1a` → `10.0.1.0/24` (AZ-1a)
* `public-subnet-1b` → `10.0.2.0/24` (AZ-1b)

### Private Subnets

* `private-subnet-1a` → `10.0.101.0/24`
* `private-subnet-1b` → `10.0.102.0/24`

Public subnets are used for load balancers, while private subnets host application servers.

---

###  Screenshots

![Subnet-1](https://github.com/user-attachments/assets/7866106e-2326-4191-b7ff-51c1a1d846db)
![Subnet-2](https://github.com/user-attachments/assets/7a410ce5-87c4-490e-b493-2267371f3bec)
![Subnet-All](https://github.com/user-attachments/assets/84de4e58-436d-4748-84e1-a1fba30385bf)

---

##  Step 3: Internet Gateway

###  Theory

An **Internet Gateway (IGW)** enables communication between resources in the VPC and the internet.

---

###  Configuration

* Name: `ecommerce-igw`
* Attach to: `ecommerce-vpc`

---

###  Screenshots

![IGW-1](https://github.com/user-attachments/assets/a5de5513-5bf2-4ec2-807a-db2ec7e1d826)
![IGW-2](https://github.com/user-attachments/assets/3ff1d7d9-c455-4435-9910-4dc4bfaab1dd)
![IGW-3](https://github.com/user-attachments/assets/144544d7-85d3-4a86-ba96-4444d283090e)

---

##  Step 4: Route Table Configuration

###  Theory

A route table defines how network traffic is directed within the VPC.

---

###  Configuration

* Route Table Name: `public-rt`
* Route Added:

```text
0.0.0.0/0 → Internet Gateway
```

* Associated Subnets:

  * `public-subnet-1a`
  * `public-subnet-1b`

This configuration enables outbound internet access for public subnets.

---

### Screenshots

![RT-1](https://github.com/user-attachments/assets/6530fac0-ac1f-49cb-8f3d-de70ca179a54)
![RT-2](https://github.com/user-attachments/assets/8a67515c-bd4a-4bfd-977b-ede0d3c0de01)
![RT-3](https://github.com/user-attachments/assets/a69dab24-689b-4605-8788-465550db71de)

---

###  Network Flow

![Flow](https://github.com/user-attachments/assets/e3086a3b-31e7-4923-bbd3-0652a9443c28)

---

##  Step 5: EC2 Instance Deployment

###  Theory

Amazon EC2 provides scalable virtual servers for hosting applications.

---

###  Configuration

* Instance Type: `t3.micro`
* Operating System: Ubuntu
* Security Group:

  * Port 22 (SSH)
  * Port 80 (HTTP)

---

###  User Data Script

Automates installation and deployment:

```bash id="c6p7l0"
#!/bin/bash
apt-get update -y
apt-get install -y nginx git

rm -rf /var/www/html/*
git clone <github-repo> /var/www/html/

chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/

systemctl restart nginx
```
###  Launch Instance
---

###  Screenshots

![EC2-1](https://github.com/user-attachments/assets/b83b943f-fba4-4db0-8887-23c379dec977)
![EC2-2](https://github.com/user-attachments/assets/ab6bc659-27e1-4323-aafd-bf1a8ec6a706)
![EC2-3](https://github.com/user-attachments/assets/0bbf33da-9d75-4e26-a259-7c4324d84f00)
![EC2-4](https://github.com/user-attachments/assets/eafc14ac-29a8-481e-9be1-f5eb67c062d1)

---
## step 6:  AMI (Amazon Machine Image) – Steps

###  Purpose

AMI is used to create identical EC2 instances with pre-installed configuration.

### Steps to Create AMI

1. Go to **EC2 Dashboard**
2. Click **Instances**
3. Select the configured instance
4. Click **Actions → Image and templates → Create Image**
5. Enter:

   * Image Name: `ecommerce-ami`
6. Click **Create Image**
7. Wait until AMI status becomes **Available**

---

###  Launch Template

###  Purpose

Launch Template is used to define instance configuration for Auto Scaling.

###  Steps to Create Launch Template

1. Go to **EC2 Dashboard**
2. Click **Launch Templates**
3. Click **Create Launch Template**

###  Configuration:

* Template Name: `ecommerce-template`
* AMI: Select created AMI
* Instance Type: `t3.micro`
* Key Pair: Select existing
* Security Group: Allow SSH (22) & HTTP (80)

###  Storage:

* Set Volume Size: **20 GB**

4. Click **Create Launch Template**

---

###  Screenshots

![LT-1](https://github.com/user-attachments/assets/4e84acff-df91-48af-a800-7c86031ae14b)
![LT-2](https://github.com/user-attachments/assets/db465207-b397-449c-93b8-7f0e4d29d80a)
![LT-3](https://github.com/user-attachments/assets/3ed651a9-dfce-4f4e-b918-c37d11c14086)

---

## Step 7: Load Balancer Configuration

###  Theory

A Load Balancer distributes incoming traffic across multiple instances to ensure availability and reliability.

---

###  Configuration

* Type: Application Load Balancer
* Scheme: Internet-facing
* Listener: HTTP (Port 80)
* Target Group: EC2 instances

---

###  Screenshots

![ALB-1](https://github.com/user-attachments/assets/0c5d3374-0121-44b0-871a-1e0ad402875e)
![ALB-2](https://github.com/user-attachments/assets/1c27e26c-dfae-4f40-bea1-c97e4844603a)
![ALB-3](https://github.com/user-attachments/assets/9095cb2f-1f20-4f70-b9df-570470d199b1)
![ALB-4](https://github.com/user-attachments/assets/fc2a2f3c-4964-4105-9d8c-2d9bcf1d7ab2)

---

##  Step 8: Auto Scaling & Monitoring

###  Theory

* **Auto Scaling** automatically adjusts the number of instances based on demand
* Amazon CloudWatch collects metrics and triggers scaling actions

---

###  Configuration

### Scaling Policies:

* Scale-Out: CPU Utilization > 70%
* Scale-In: CPU Utilization < 30%

---

###  Screenshots

![ASG](https://github.com/user-attachments/assets/638fa614-b550-4041-8f59-89456c22bd36)
![Metrics](https://github.com/user-attachments/assets/2e191870-a355-4b29-b942-a9f0d184ccdd)
![Scale-Out](https://github.com/user-attachments/assets/7956a950-651b-4813-b8d0-6a7d59612e79)
![Scale-In](https://github.com/user-attachments/assets/808410e5-b96f-4a1a-b9fc-cc673bc68674)
<img width="1908" height="829" alt="Screenshot 2026-03-30 154433" src="https://github.com/user-attachments/assets/95f8abc5-a95f-43bb-af17-d5c951c3d16d" />

---

##  Step 9: Health Checks

###  Theory

Health checks ensure that only healthy instances receive traffic.

---

###  Configuration

* Protocol: HTTP
* Path: `/`

---

###  Screenshot

![Health](https://github.com/user-attachments/assets/1c4f8da8-35a8-4293-b9b0-8028b1405878)

---

###  Final Output

![Final](https://github.com/user-attachments/assets/88387be4-8206-4d50-a492-667f069434d5)

Application is accessed via the Load Balancer DNS.

---

## Step 10: S3 Storage

### Theory

Amazon S3 is used for storing static assets such as images, CSS, and JavaScript files.

---

###  Configuration

* Versioning: Enabled
* Encryption: SSE-S3

---

###  Screenshots

<img width="1910" height="795" alt="Screenshot 2026-03-30 182045" src="https://github.com/user-attachments/assets/ab28673d-fb27-48c2-b2a6-6b3452f68b29" />
<img width="1888" height="257" alt="Screenshot 2026-03-30 182057" src="https://github.com/user-attachments/assets/d1692c42-a27f-42ed-ad2f-1b3e137aa6eb" />
<img width="1891" height="404" alt="Screenshot 2026-03-30 182111" src="https://github.com/user-attachments/assets/48924e33-5c14-4eb2-8490-957db323783c" />

---

##  Conclusion

The **Highly Available 3-Tier Web Application on AWS** demonstrates a scalable and reliable cloud architecture. By utilizing Amazon EC2, Amazon S3, and Amazon CloudWatch, the system efficiently handles dynamic workloads.

The use of a custom VPC with public and private subnets improves security, while the Application Load Balancer and Auto Scaling ensure consistent performance and availability.

Overall, the architecture achieves high availability, fault tolerance, and scalability, making it suitable for real-world applications.



















