# NexusCart E-Commerce Centralized Log Analysis & Security Auditing

## Overview
This project simulates a typical web application built around an n-tier architecture for an e-commerce platform, NexusCart. The objective is to centralize log collection across compute, network, and database layers to inspect potential security vulnerabilities using Amazon CloudWatch Logs Insights.

___
## Problem Statement

NexusCart operates a distributed e-commerce application spread across Linux and Windows web servers, an RDS database, and multiple subnets. Tracking security threats, such as unauthorized SSH access attempts or illegitimate database connections, is highly inefficient when logs are decentralized and siloed across individual servers and services.

___
## Problem Reframing & Requirements

To proactively identify and respond to security threats, the infrastructure requires:
* Automated, centralized log collection from EC2 instances without manually SSH/RDPing into servers.
* Visibility into network traffic flows across the VPC to identify rejected packets and unauthorized port access.
* Database error logging to track failed connection attempts.
* A powerful querying mechanism to instantly filter and aggregate log data across all these sources.

___
## Solution & Tool Tradeoffs

The chosen solution leverages Amazon CloudWatch as the centralized logging and analytics hub.

**Tradeoffs Considered:**
* **AWS Systems Manager vs. Manual Agent Installation:** Using Systems Manager Run Command and Parameter Store to deploy the CloudWatch Agent allows for fleet-wide configuration at scale without manual intervention.
* **CloudWatch Logs Insights vs. Third-Party SIEM:** CloudWatch Logs Insights provides a purpose-built query language integrated natively with AWS services, eliminating the data transfer costs and operational overhead of exporting logs to an external SIEM for real-time operational troubleshooting.

___
## Architecture
<img width="1214" height="903" alt="architecture" src="https://github.com/user-attachments/assets/b2c2db79-33a8-4e88-8310-03fb40912681" />

* Compute: Amazon EC2 (Linux/Apache and Windows/IIS) acting as web servers.
* Database: Amazon RDS (MySQL engine).
* Network: Amazon VPC with public and private subnets, utilizing VPC Flow Logs.
* Management & Monitoring: AWS Systems Manager, Amazon CloudWatch Logs, and CloudWatch Logs Insights.

___
## Data Assets & Schemas

* **AccessLogGroup & IISLogGroup:** Captures HTTP access patterns, status codes (e.g. 404, 200), and source IP addresses from the web servers.
* **VPCFlowLogGroup:** Captures network interfaces, source/destination IPs, ports, and ACCEPT/REJECT actions.
* **RDS Error Logs:** Captures database-level errors and connection failures.
* **SSHLogGroup / CloudTrailLogGroup:** Captures SSH access attempts and AWS API usage by IAM users.

___
## Pipeline Execution Flow
<img width="1920" height="1080" alt="setupCloudWatchAgents" src="https://github.com/user-attachments/assets/5cd45229-97c3-49df-b184-ffb605ed403a" />

1. **Agent Deployment:** Assigned Systems Manager IAM policies to EC2 instances and used Systems Manager Run Command to install and start the Amazon CloudWatch Agent using configurations stored in Parameter Store.
2. **Network Logging:** Configured Amazon VPC Flow Logs to capture all network traffic across the VPC and route it to a dedicated CloudWatch Log Group.
3. **Database Logging:** Modified the Amazon RDS MySQL instance to publish Error logs directly to CloudWatch.
4. **Log Analysis:** Utilized CloudWatch Logs Insights queries to identify illegitimate database calls (traffic not on port 3306), find the top 20 source IPs generating rejected requests, and track SSH login attempts.
5. **Metric Filters:** Created CloudWatch Metric Filters to track occurrences of "404" errors in the Apache and IIS logs and visualized them using CloudWatch Metrics.

___
## Security & Roles

* IAM Roles for EC2: Instances were attached with Simple Systems Manager (SSM) policies to allow Systems Manager to manage them securely.
* VPC Flow Logs Role: A dedicated IAM role was utilized with trust relationships allowing the flow logs service to publish data to CloudWatch Logs.

___
## Business Outcomes & Analytical Outputs

* Centralized Security Auditing: Successfully aggregated network, compute, and database logs into a single pane of glass, enabling rapid threat detection.
* Actionable Insights: Used interactive queries to successfully uncover suspicious network behavior, such as targeted attacks on non-standard database ports and unauthorized SSH access attempts.
* Real-Time Dashboards: Built metric filters that convert unstructured log data (404 errors) into quantitative metrics that can trigger operational alarms.
