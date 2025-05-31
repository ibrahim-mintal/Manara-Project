#  Scalable Web Application with Autoscaling and ALB

## Overview

This project demonstrates a scalable and highly available AWS architecture designed for a secure web application. The infrastructure spans across two Availability Zones and includes VPC, public/private subnets, Application Load Balancer (ALB), Auto Scaling Groups (ASG), NAT Gateway, RDS (MySQL) with Multi-AZ, and monitoring via CloudWatch and SNS.

---

## 🖐️ Project Architecture
![Architecture Diagram](Architecture-Project-1.svg)





## Architecture Components

1.  **Route 53:**
    *   AWS managed DNS service. Routes user requests from the internet to the appropriate AWS resource, in this case, the public-facing Web Tier ALB.

2.  **VPC (Virtual Private Cloud):**
    *   Region: `us-east-1`
    *   Availability Zones: `us-east-1a` (AZ1), `us-east-1b` (AZ2)
    *   Provides an isolated network environment.

3.  **Internet Gateway (IGW):**
    *   Attached to the VPC, enabling communication between instances in public subnets and the internet.

4.  **NAT Gateway:**
    *   Located in `public subnet 2` (AZ2).
    *   Allows instances in private subnets to initiate outbound traffic to the internet (e.g., for software updates) while preventing inbound traffic initiated from the internet.

5.  **Subnets:**
    *   **Public Subnets:**
        *   `public subnet 1` (AZ1): `192.168.1.0/24` - Contains Bastion Host.
        *   `public subnet 2` (AZ2): `192.168.2.0/24` - Contains NAT Gateway.
    *   **Private Subnets (Web Tier):**
        *   `private subnet 1` (AZ1): `192.168.3.0/24` - Hosts Web Server instances.
        *   `private subnet 2` (AZ2): `192.168.4.0/24` - Hosts Web Server instances.
    *   **Private Subnets (App Tier):**
        *   `private subnet 3` (AZ1): `192.168.5.0/24` - Hosts App Server instances.
        *   `private subnet 4` (AZ2): `192.168.6.0/24` - Hosts App Server instances.
    *   **Private Subnets (Database Tier):**
        *   `private subnet 5` (AZ1): `192.168.7.0/24` - Hosts primary RDS instance.
        *   `private subnet 6` (AZ2): `192.168.8.0/24` - Hosts standby RDS instance.

6.  **Route Tables:**
    *   **Public Route Table:** Associated with public subnets. Routes internet-bound traffic (`0.0.0.0/0`) to the IGW.
    *   **Private Route Tables (1, 2, 3):** Associated with respective private subnets. Routes internet-bound traffic (`0.0.0.0/0`) to the NAT Gateway.

7.  **Bastion Host:**
    *   Located in `public subnet 1`.
    *   Provides a secure jump point for administrators to access instances in private subnets (e.g., via SSH on port 22). Port 80 is also indicated, potentially for a web-based admin tool or health check.

8.  **Application Load Balancers (ALB):**
    *   **ALB (web):** Public-facing, receives traffic from the internet via Route 53 and the IGW, and distributes it across the Web Server instances managed by the Web-Tier ASG.
    *   **ALB (App):** Internal, receives traffic from the Web Servers and distributes it across the App Server instances managed by the App-Tier ASG.

9.  **Auto Scaling Groups (ASG):**
    *   **Web-Tier ASG:** Manages EC2 instances running the web server application, scaling in/out based on defined policies (e.g., CPU utilization). Instances are launched in `private subnet 1` and `private subnet 2`.
    *   **App-Tier ASG:** Manages EC2 instances running the application server logic, scaling in/out based on defined policies. Instances are launched in `private subnet 3` and `private subnet 4`.

10. **EC2 Instances:**
    *   **Web Servers:** Handle incoming HTTP requests, potentially serving static content and forwarding dynamic requests to the App Tier.
    *   **App Servers:** Process application logic, interact with the database, and return responses to the Web Tier.

11. **IAM Role:**
    *   Associated with the App Server instances.
    *   Grants necessary permissions for the application servers to interact with other AWS services (RDS).

12. **RDS (Relational Database Service):**
    *   **Database Engine:** MySQL
    *   **Deployment:** Multi-AZ configuration with a primary instance in `private subnet 5` (AZ1) and a standby instance in `private subnet 6` (AZ2) for high availability and automatic failover.
    *   **RDS Policy:** Associated with the RDS instance, likely defining access control, backup strategies, or encryption settings.

13. **CloudWatch:**
    *   Monitors metrics for various AWS resources (EC2 instances, ALBs, RDS, ASGs, Route 53 health checks).
    *   Used to trigger alarms (e.g., for scaling ASGs) and provide operational visibility.

14. **SNS (Simple Notification Service):**
    *   Used to send notifications based on CloudWatch alarms or other events (e.g., ASG scaling activities, Route 53 health check failures).
    *   Configured to send E-mail notifications to administrators.

## Network Configuration Details

*   Route 53 resolves the application domain name to the public ALB's DNS name.
*   The VPC provides network isolation.
*   Public subnets have direct internet access via the IGW.
*   Private subnets rely on the NAT Gateway for outbound internet connectivity.
*   Traffic flow:
    *   Users -> Internet -> Route 53 -> IGW -> ALB (web) -> Web Servers (ASG in private subnets 1 & 2)
    *   Web Servers -> ALB (App) -> App Servers (ASG in private subnets 3 & 4)
    *   App Servers -> RDS MySQL (private subnets 5 & 6)
    *   Private Instances -> NAT GW -> IGW -> Internet (for outbound connections)
*   Route tables ensure proper traffic routing between subnets, the IGW, and the NAT Gateway.

## Security Considerations

*   **DNS Security:** Route 53 offers features like DNSSEC for added security (configuration optional).
*   **Network Segmentation:** Public and private subnets separate resources based on exposure requirements.
*   **Bastion Host:** Provides controlled administrative access to private resources.
*   **NAT Gateway:** Allows private instances outbound access without direct inbound exposure.
*   **Security Groups (Implied):** Essential to control traffic flow between tiers (e.g., allowing traffic from ALB-web to Web Servers, Web Servers to ALB-App, App Servers to RDS).
*   **IAM Roles:** Used for secure service-to-service communication for App Servers.
*   **RDS Policies:** Enhance database security.

## Scalability and High Availability

*   **Route 53:** Can be configured with health checks and routing policies (e.g., latency-based, failover) to improve availability and performance.
*   **ALBs:** Distribute load across multiple instances within each tier.
*   **ASGs:** Automatically adjust the number of Web and App server instances based on load, ensuring performance and cost-efficiency.
*   **Multi-AZ Deployment:** Spreading instances across two AZs (us-east-1a, us-east-1b) mitigates the impact of a single AZ failure.
*   **RDS Multi-AZ:** Provides data redundancy and automatic failover for the database.

## Monitoring and Notifications

*   **CloudWatch:** Provides essential monitoring of resource health and performance, including Route 53 health checks.
*   **SNS:** Enables proactive notifications to administrators regarding important events or alarms.

## Conceptual Setup Steps

Below are the high-level steps to provision this architecture. Detailed configuration (e.g., specific Security Group rules, ASG policies, IAM permissions, Route 53 records) would be required for a functional deployment.

1.  **VPC Setup:**
    *   Create a VPC in the `us-east-1` region.
    *   Create six subnets across two AZs (`us-east-1a`, `us-east-1b`) using the specified CIDRs: `192.168.1.0/24`, `192.168.2.0/24` (public); `192.168.3.0/24`, `192.168.4.0/24` (private web); `192.168.5.0/24`, `192.168.6.0/24` (private app); `192.168.7.0/24`, `192.168.8.0/24` (private db).
2.  **Internet Access:**
    *   Create an Internet Gateway (IGW) and attach it to the VPC.
    *   Create a NAT Gateway in one of the public subnets (e.g., `public subnet 2`). Allocate an Elastic IP for it.
3.  **Routing:**
    *   Create a Public Route Table, associate it with the public subnets, and add a default route (`0.0.0.0/0`) pointing to the IGW.
    *   Create Private Route Tables (one for each tier or AZ pair), associate them with the respective private subnets, and add a default route (`0.0.0.0/0`) pointing to the NAT Gateway.
4.  **Security Groups:**
    *   Create Security Groups for Bastion Host (allow SSH from trusted IPs,HTTP).
    *   Create Security Group for Web Tier ALB (allow HTTP from internet).
    *   Create Security Group for Web Servers (allow traffic from Web Tier ALB SG, allow outbound to App Tier ALB SG).
    *   Create Security Group for App Tier ALB (allow traffic from Web Servers SG).
    *   Create Security Group for App Servers (allow traffic from App Tier ALB SG, allow outbound to RDS SG and NAT GW).
    *   Create Security Group for RDS (allow traffic from App Servers SG on the MySQL port).
5.  **Bastion Host:**
    *   Launch an EC2 instance (e.g., t2.micro Linux) in `public subnet 1`.
    *   Assign the Bastion Security Group.
6.  **IAM Role & Policy:**
    *   Create an IAM Role for the EC2 instances in the App-Tier ASG with necessary permissions.
    *   Define an RDS Policy (details depend on specific requirements).
7.  **RDS Database:**
    *   Create an RDS Subnet Group using the database private subnets (`private subnet 5`, `private subnet 6`).
    *   Launch a Multi-AZ MySQL RDS instance, placing it within the RDS Subnet Group and assigning the RDS Security Group.
    *   Apply the RDS Policy if applicable.
8.  **Launch Configurations/Templates:**
    *   Create a Launch Configuration or Launch Template for the Web Servers (including application code, web server software, agent configurations).
    *   Create a Launch Configuration or Launch Template for the App Servers (including application code, necessary libraries, agent configurations, assign IAM Role).
9.  **Auto Scaling Groups & Load Balancers:**
    *   Create the internal Application Load Balancer (ALB-App) targeting the App Tier private subnets (`private subnet 3`, `private subnet 4`). Assign the App Tier ALB Security Group.
    *   Create the App-Tier ASG using its Launch Configuration/Template, configured to launch instances in `private subnet 3` & `private subnet 4`. Attach it to the ALB-App Target Group.
    *   Create the public Application Load Balancer (ALB-Web) targeting the Web Tier private subnets (`private subnet 1`, `private subnet 2`). Assign the Web Tier ALB Security Group.
    *   Create the Web-Tier ASG using its Launch Configuration/Template, configured to launch instances in `private subnet 1` & `private subnet 2`. Attach it to the ALB-Web Target Group.
10. **Monitoring & Notifications:**
    *   Create CloudWatch Alarms based on relevant metrics (e.g., CPU Utilization for ASG scaling, RDS metrics, ALB health, Route 53 health checks).
    *   Create an SNS Topic for notifications.
    *   Configure CloudWatch Alarms and other services (e.g., ASG notifications, Route 53 health checks) to publish to the SNS Topic.
    *   Subscribe an administrator's email address to the SNS Topic.
11. **DNS (Route 53):**
    *   Create a Hosted Zone in Route 53 for your domain.
    *   Create an Alias record (e.g., 'www.yourdomain.com' or 'app.yourdomain.com') pointing to the DNS name of the public-facing ALB (web).
