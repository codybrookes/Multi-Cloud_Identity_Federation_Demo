# Multi-Cloud_Identity_Federation_Project
Demonstrates the difference between federation and SSO between platform(Systems - AWS + Azure) 
**Technologies:** AWS, Azure AD, Python, Lambda, S3, SES, CloudTrail, CloudWatch

## Overview
This project demonstrates multi-cloud identity federation between Azure AD and AWS. Users authenticate with Azure credentials to access AWS resources securely. The system automates IAM user provisioning in AWS, sends email notifications, and monitors all identity events.

## Features
- SAML-based federation: Azure AD → AWS IAM
- Automated user and role provisioning from CSV
- Least-privilege policy enforcement
- SES notifications on new users
- CloudTrail logging and CloudWatch monitoring

## Architecture
Azure AD → SAML → AWS IAM Roles
CSV → Lambda → IAM APIs → SES notifications
CloudTrail → S3 & CloudWatch → Alarms/SNS

### Prerequisites
- AWS account with admin privileges
- Azure AD tenant
- Python 3.9+ and pip
- AWS CLI configured
- Optional: AWS SAM CLI

### Steps
1. **Azure AD setup**
   - Register enterprise app and configure SAML SSO
   - Assign users or groups
   - Download metadata XML

2. **AWS setup**
   - Create SAML IdP with Azure metadata
   - Create IAM roles for federated access

3. **Automate AWS IAM**
   - Upload `users.csv` to S3
   - Deploy `iam_federation.py` Lambda
   - Configure environment variables
   - Attach minimal IAM role to Lambda

4. **Email Notifications**
   - Verify `SES_FROM` email in SES
   - Test notifications via Lambda

5. **Monitoring & Auditing**
   - Enable CloudTrail
   - Configure CloudWatch metric filters and alarms
   - Optionally, archive logs to S3/Glacier

## Security Best Practices
- Prefer temporary credentials and centralized identity
- Avoid long-lived access keys unless necessary
- Least-privilege policies for Lambda and IAM roles
- Keep CloudTrail logs in a secure S3 bucket

## Files in this Repo
- `iam_federation.py` — Lambda function script
- `lambda-policy.json` — Least-privilege policy for Lambda
- `template.yaml` — Optional SAM template
- `users.csv` — Example input file
- `README.md` — This file
