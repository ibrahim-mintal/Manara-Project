# 🚀 Scalable Web Application on AWS with ALB and Auto Scaling

## 📖 Overview

This project demonstrates a scalable and highly available AWS architecture designed for a secure web application. The infrastructure spans across two Availability Zones and includes VPC, public/private subnets, Application Load Balancer (ALB), Auto Scaling Groups (ASG), NAT Gateway, RDS (MySQL) with Multi-AZ, and monitoring via CloudWatch and SNS.

---

## 🖐️ Project Architecture
![Architecture Diagram](Architecture-Project-1.svg)


### Key Components:

* **VPC**: Isolated virtual network environment.
* **Public Subnets**: Host Bastion Host and NAT Gateway.
* **Private Subnets**: Host EC2 instances and RDS.
* **ALB**: Distributes incoming web traffic to EC2 instances across AZs.
* **ASG**: Dynamically scales EC2 instances based on CPU usage.
* **NAT Gateway**: Allows private instances outbound internet access.
* **Bastion Host**: SSH access point for private EC2s.
* **RDS**: Multi-AZ MySQL database for backend storage.
* **CloudWatch + SNS**: Performance monitoring and alerting.

---

## 🛠️ Step-by-Step Deployment Guide

### 1. Set Up VPC

* Create a VPC with CIDR block: `192.168.0.0/16`
* Add an Internet Gateway (IGW)

### 2. Create Subnets

* **Public Subnets**:

  * Subnet 1: `192.168.1.0/24` (us-east-1a)
  * Subnet 2: `192.168.2.0/24` (us-east-1b)
* **Private Subnets**:

  * App: `192.168.3.0/24`, `192.168.4.0/24`
  * DB: `192.168.5.0/24`, `192.168.6.0/24`

### 3. Configure Route Tables

* Public subnet → IGW (0.0.0.0/0)
* Private subnets → NAT Gateway (0.0.0.0/0)

### 4. Deploy NAT Gateway

* Create an Elastic IP
* Attach NAT Gateway in public subnet 2
* Update private subnet route tables

### 5. Set Up EC2 Instances

* Launch EC2 instances in private subnets
* Configure Auto Scaling Group (ASG)
* Attach an IAM Role with necessary permissions

### 6. Configure Application Load Balancer

* ALB in public subnets
* Target group: EC2 instances
* Listener: Port 80 (HTTP)

### 7. Set Up RDS (MySQL)

* Launch RDS in DB subnets with Multi-AZ enabled
* Attach RDS Security Group with port 3306 access from EC2 SG

### 8. Monitoring and Alerts

* Create CloudWatch Alarms for:

  * CPU utilization on EC2
  * RDS storage or CPU
* Use SNS topic for email alerts

---

## 🔐 Accessing the Instances

* **Bastion Host**:

  * SSH into the bastion using public IP and your private key
* **Private EC2s**:

  * SSH from the bastion host using private IPs

---

## 🛡️ Security Considerations

* Use Security Groups to restrict traffic (Least Privilege)
* Limit SSH access to Bastion IPs only
* Enable CloudTrail and logging for auditing
* Enforce IAM policies for minimal required access

---

## ✅ Best Practices Followed

* High Availability: Multi-AZ deployments for both EC2 and RDS
* Scalability: Auto Scaling Group configured
* Fault Tolerance: Load Balancer and NAT Gateway redundancy
* Monitoring: CloudWatch with SNS for proactive alerting
* Cost Optimization: Instances scale based on demand
