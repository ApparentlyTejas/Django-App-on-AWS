# Django Blog Page Application deployed on AWS ALB with Auto Scaling, S3, RDS, VPC, Lambda, DynamoDB, CloudFront + Route 53

## Description

As part of my AWS capstone project for Masters, I deployed a Django blog app on AWS infrastructure. Used Application Load Balancer with Auto Scaling EC2 instances + RDS MySQL database all inside custom VPC. Added CloudFront and Route 53 at front for secure access. Users can register, write blog posts, upload pictures/videos to S3 (tracked in DynamoDB via Lambda).

## Project Details

![Project](project.jpg)

**Project requirements:**
- Deploy Django blog app in isolated VPC 
- User registration → RDS MySQL (private subnet)
- Pictures/videos → S3 bucket
- S3 objects tracked in DynamoDB via Lambda trigger
- Secure browser access from anywhere
- Push code to GitHub, pull into production EC2s

**VPC Specs:**
- 2 AZs, each with 1 public + 1 private subnet
- Internet Gateway attached
- NAT instance in public subnet (also bastion)
- Separate public/private route tables

**ALB + ASG:**
Auto Scaling Group:

Launch Template based

Desired: 2, Min: 2, Max: 4

Health check: ELB (90s grace)

Target tracking: 70% CPU, 200s warmup

Email notifications for scaling events

ALB:

SG allows HTTP/HTTPS (80/443) from anywhere

ACM cert, HTTP→HTTPS redirect

Target group HTTP health checks

text

**Launch Template:**
- Ubuntu 18.04 t2.micro
- User data: install Django, pull GitHub code, deploy port 80
- SG: HTTP/HTTPS from ALB only + SSH(22)
- IAM role: S3 full access
- Tag: "AWS Capstone Project"

**RDS:**
- MySQL 8.0.20 db.t2.micro (private subnet)
- Only EC2 via ALB SG on port 3306
- Endpoint in Django settings.py

**CloudFront:**
- Cache in front of ALB (HTTPS origin)
- Redirect HTTP→HTTPS
- All methods + forward all cookies
- ACM cert (same as ALB)

**Route 53 Failover:**
- Primary: CloudFront
- Secondary: S3 static "under construction" page
- Health check CloudFront

**S3 Buckets:**
1. Main bucket (VPC region) + VPC endpoint (private EC2 access)
2. Failover static bucket

**Lambda + DynamoDB:**
- Python 3.8, S3 trigger
- Writes S3 objects to DynamoDB (PK: id)
- IAM: S3/DynamoDB full access + VPC network

## Expected Outcome

![Demo](outcome.png)

## Topics I Implemented

- Bash scripting (user data)
- VPC (subnets, RTs, IGW, NAT, endpoints)
- ALB + Target Groups + Listeners
- ASG + Launch Templates
- RDS MySQL
- Security Groups (ALB→EC2→RDS)
- IAM roles
- S3 + static sites
- Lambda + S3 events + DynamoDB
- ACM certs, CloudFront, Route 53 failover
- GitHub deployment

## Solution Steps (What I Did)

1. Created VPC + all networking
2. Security Groups (ALB→EC2→RDS chain)
3. RDS in private subnet
4. 2 S3 buckets (main + failover static)
5. Pushed Django code to GitHub
6. Wrote Launch Template user data script
7. Updated Django settings.py (RDS/S3)
8. NAT instance (bastion)
9. Launch Template + IAM role
10. ACM certificate
11. ALB + Target Group
12. ASG with scaling policy
13. CloudFront distribution
14. Route 53 failover routing
15. DynamoDB table (PK: id)
16. Lambda function + S3 trigger

## Notes

- RDS private only - EC2 via SG on 3306
- ALB public - redirects HTTP→HTTPS
- EC2 private - ALB traffic only
- S3 via VPC endpoint (no public internet)

## Resources

- [Django](https://www.djangoproject.com/)
- [Django Tutorial](https://realpython.com/get-started-with-django-1/)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/index.html)
