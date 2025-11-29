Django Blog Application on AWS
Django blog web application deployed on AWS using Application Load Balancer, Auto Scaling, S3, RDS, VPC components, Lambda, DynamoDB, CloudFront, and Route 53.

Description
This project deploys a Django blog application on AWS with a fully isolated VPC and a multi‑tier architecture. The setup uses an Application Load Balancer in front of an Auto Scaling Group of EC2 instances, backed by an RDS MySQL database in private subnets. CloudFront and Route 53 sit at the edge to handle secure, global access, and static media (images/videos) are stored in S3. S3 object metadata is tracked in DynamoDB via a Lambda function.

Users can register, create blog posts, and upload images/videos. User data is stored in RDS, media files go to S3, and all traffic is routed securely through HTTPS with proper network isolation between public, private, and database tiers.

Project Details
The goal is to host a Django blog application in its own VPC with a clear separation between web, application, and database layers.

The application is developed by a full‑stack team; this setup focuses on the infrastructure, deployment, and security on AWS.

User registration data is stored in an RDS MySQL database, media files are stored in S3, and S3 object listings (photos/videos) are written to a DynamoDB table.

The application is deployed using the Django framework and must be reachable securely from any browser.

Source code is kept in GitHub, and the production EC2 instances pull the application from the repository.

Infrastructure and VPC

All resources are created as new AWS components for this stack.

VPC design:

2 Availability Zones.

Each AZ has 1 public and 1 private subnet.

An Internet Gateway is attached to the VPC.

One public subnet hosts a NAT instance (optionally also used as a bastion host).

Separate route tables for public and private subnets.

Route tables and subnet associations are configured according to public/private routing policies.

Compute and Load Balancing

Application Load Balancer + Auto Scaling Group of Ubuntu 18.04 EC2 instances.

ALB:

Placed in a security group that allows HTTP (80) and HTTPS (443) from anywhere.

Uses an ACM certificate for HTTPS.

Redirects HTTP to HTTPS.

Target group uses HTTP for health checks.

Auto Scaling Group:

Uses a Launch Template to define EC2 configuration.

Uses all AZs in the VPC.

Desired capacity: 2 instances.

Min size: 2, Max size: 4.

Health check type: ELB, grace period: 90 seconds.

Scaling Policy: Target tracking on average CPU utilization at 70% with 200 seconds warm‑up.

Notifications configured for instance launch/terminate/fail events.

Launch Template

Prepares the Django environment based on developer notes.

Deploys the Django app on port 80.

Security group:

Allows HTTP (80) and HTTPS (443) only from the ALB security group.

Allows SSH (22) from allowed IPs (original text says “from anywhere”, adjust as needed).

EC2 type: t2.micro.

Instances are tagged as AWS Capstone Project.

IAM role attached to EC2 with full S3 access so Django can talk to S3.

Database (RDS)

RDS instance in a private subnet.

Engine: MySQL 8.0.20.

Instance type: db.t2.micro.

RDS endpoint is configured in Django settings as per developer notes.

Only EC2 instances (via correct security groups) can reach RDS on port 3306.

CloudFront and Route 53

CloudFront:

Used as a cache layer in front of the ALB.

Origin Protocol Policy: HTTPS only.

Viewer Protocol Policy: Redirect HTTP to HTTPS.

Cache behavior:

Allows GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE.

Forwards all cookies.

Uses the ACM certificate (can be the same as ALB).

Route 53:

Public hosted zone with a custom hostname.

Uses HTTPS for access.

Failover routing policy:

Primary: CloudFront distribution.

Secondary: Static website hosted on a separate S3 bucket that shows “page is under construction”.

Health check monitors CloudFront; Route 53 fails over to the static S3 site if CloudFront is unhealthy.

S3 Buckets

First S3 Bucket:

Created in the same region as the VPC.

VPC endpoint configured to avoid exposing S3 ↔ EC2 traffic to the public internet.

Bucket name is referenced in the Django configuration as per developer notes.

Second S3 Bucket:

Used as the failover static website for Route 53.

Hosts a simple static page with an “under construction” image.

Lambda + DynamoDB

Lambda:

Runtime: Python 3.8.

Function code stored in GitHub.

Triggered by S3 events from the first bucket.

Runs inside the VPC and needs:

S3 full access.

DynamoDB full access.

Appropriate network permissions (e.g., NetworkAdministrator policy).

S3 event:

Created on the first S3 bucket to trigger the Lambda on object operations.

DynamoDB:

Table with primary key id.

Table name is configured inside the Lambda function.

Stores metadata for objects in the S3 bucket.

Expected Outcome
Technologies / Topics Involved

Bash scripting

EC2 Launch Templates

VPC (subnets, route tables, routing, IGW, NAT, bastion, endpoints)

Application Load Balancer (ALB), target groups, listeners

Auto Scaling Groups

RDS (MySQL)

Security Groups for EC2, RDS, ALB

IAM roles and policies

S3 configuration and static website hosting

DynamoDB tables

Lambda functions and event triggers

ACM certificates

CloudFront distributions

Route 53 DNS and routing

Git & GitHub for version control

Skills Demonstrated
Designing and building a VPC with public/private subnets, route tables, NAT, and secure routing.

Deploying and configuring a Django application with RDS MySQL as the backend.

Using user data in a Launch Template to bootstrap EC2 instances with the app stack.

Building a Lambda + DynamoDB integration triggered by S3 events.

Wiring together EC2, Launch Templates, ALB, ASG, S3, RDS, CloudFront, and Route 53 into a working end‑to‑end architecture.

Using Git and GitHub (push, pull, commit, branching) as the VCS.

Solution Steps (High Level)
Create a dedicated VPC and all networking components.

Create security groups for ALB → EC2 → RDS.

Create the RDS instance in a private subnet.

Create two S3 buckets and enable static website hosting on the failover bucket.

Download/clone the project code.

Prepare your GitHub repository.

Write user data for the Launch Template to install and configure the Django app.

Configure RDS and S3 settings in Django configuration.

Create a NAT instance in the public subnet.

Create the Launch Template and IAM role.

Request an ACM certificate for HTTPS.

Create the Application Load Balancer and target group.

Create the Auto Scaling Group based on the Launch Template.

Create CloudFront in front of the ALB.

Configure Route 53 with failover routing.

Create the DynamoDB table.
17–18. Create the Lambda function and S3 event trigger.

Notes
RDS runs in private subnets and is only reachable from EC2 instances via the appropriate security groups and port 3306.

ALB sits in public subnets and redirects HTTP to HTTPS.

EC2 instances sit in private subnets and only accept traffic from the ALB.

S3 traffic from EC2 is kept inside the VPC via an endpoint.

Resources
Django

Django Getting Started

AWS CLI Reference
