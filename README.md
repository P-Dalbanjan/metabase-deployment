# Metabase High Availability AWS Deployment

This repository contains the Infrastructure as Code (IaC) and configuration management scripts for a highly available, secure, and self-healing deployment of Metabase on AWS. The infrastructure is primarily provisioned using Terraform, with instance configuration managed by Ansible. 

The deployment is designed to query a business database (Odoo read replica) residing in a separate VPC via VPC Peering.

---

# 🏗 Architecture Overview

The architecture is designed for fault tolerance, scalability, and strict security using AWS best practices. It spans two Availability Zones (AZs) to ensure high availability.

![Architecture Overview](assets/architecture-diagram.png)

### Key Components

* **Networking:** A custom Virtual Private Cloud (VPC) partitioned into Public and Private subnets across two AZs. 
* **Routing & Ingress:** Amazon Route 53 handles DNS, routing external HTTPS traffic to an Application Load Balancer (ALB). SSL/TLS certificates are managed by AWS Certificate Manager (ACM).
* **Compute:** Metabase runs on Amazon EC2 instances managed by an Auto Scaling Group (ASG) located strictly within private subnets.
* **Primary Database:** Amazon RDS (PostgreSQL/MySQL) configured for a Multi-AZ deployment ensures data redundancy and failover capabilities.
* **External Data Source (Odoo):** A VPC Peering connection links the Metabase VPC to an external "Odoo VPC," allowing Metabase EC2 instances to securely query an Odoo Read Replica.
* **Security & Management:**
    * **AWS Systems Manager (SSM):** Provides secure, agent-based instance access without requiring bastion hosts or open inbound SSH ports.
    * **AWS Secrets Manager:** Securely stores and rotates database credentials.
    * **IAM Roles:** EC2 instances assume IAM roles scoped with least-privilege permissions.
    * **Security Groups:** Strictly control inbound and outbound traffic at the instance and load balancer levels.
* **Backups:** Automated RDS snapshots and backups are stored in Amazon S3.

## 🚦 Traffic Flow

1.  **User Access:** Users navigate to the Metabase URL. Route 53 resolves the domain to the ALB.
2.  **Load Balancing:** The ALB terminates the SSL/TLS connection and forwards the traffic to the Target Group containing the active Metabase EC2 instances.
3.  **Application Tier:** The EC2 instances process the requests. 
    * *Internal Data:* Metabase queries its own configuration data from the Multi-AZ RDS instance.
    * *Analytics Data:* For reporting, Metabase routes queries through the VPC Peering connection to the Odoo Read Replica database.
4.  **Outbound Internet:** EC2 instances access the internet (for updates or external API calls) via NAT Gateways located in the public subnets.

## 📋 Prerequisites

Before deploying this architecture, ensure you have:
* An active AWS Account with appropriate administrative permissions.
* A registered domain name managed by Route 53 (or pointing to Route 53 nameservers).
* An existing Odoo VPC with a configured Read Replica, ready to accept a VPC Peering request.

## 🔒 Security Posture

* **Private Compute:** No EC2 instances or RDS databases have public IP addresses.
* **Encryption:** Traffic is encrypted in transit via ACM on the ALB. Data at rest is encrypted via AWS KMS for RDS and S3.
* **Access Control:** All infrastructure access is handled via AWS SSM; no SSH keys are distributed.

## 🚀 Deployment

*(Note: Add your specific deployment instructions here, such as Terraform commands, CloudFormation stack details, or AWS CDK deployment steps.)*

1.  **Network Provisioning:** Deploy the VPC, Subnets, Internet Gateway, and NAT Gateways.
2.  **VPC Peering:** Establish and accept the VPC peering connection with the Odoo VPC. Update route tables in both VPCs.
3.  **Database Provisioning:** Deploy the Multi-AZ RDS instance and store credentials in Secrets Manager.
4.  **Compute Provisioning:** Create the ALB, Target Group, ASG, and Launch Template for the EC2 instances. Ensure the Metabase user data script automatically fetches database credentials on boot.
5.  **DNS & SSL:** Request the ACM certificate and map the Route 53 alias record to the ALB.
