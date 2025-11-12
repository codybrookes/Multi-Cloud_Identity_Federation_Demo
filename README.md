# Multi-Cloud Identity Federation Project
Demonstrates the difference between federation and SSO between platform(Systems - AWS + Azure) 
**Technologies:** AWS, Azure AD, CloudTrail, CloudWatch

Project Summary

This project demonstrates the deployment and security hardening of a robust, production-ready identity federation solution, enabling users authenticated in Azure Active Directory (IdP) to seamlessly access Amazon Web Services (SP) resources using SAML 2.0.

The core goal was to implement centralized identity control and enforce Zero Trust principles, ensuring that all access to AWS is conditional, temporary, and auditable.

# Key Achievements

SAML 2.0 Implementation: Established mutual trust between Azure AD and AWS IAM.

Centralized MFA Enforcement: Implemented Azure Conditional Access to require Multi-Factor Authentication (MFA) for all AWS access attempts, centralizing security policy.

Least Privilege: Users assume the AzureAD_ReadOnlyRole, granting only the necessary permissions.

Troubleshooting: Successfully resolved a common Azure AD SAML assertion quoting error by using an attribute-based workaround (due to plan limitations), ensuring the ARN string was passed unquoted to the AWS Security Token Service (STS).

Auditability: Integrated CloudTrail and Azure Sign-in Logs for end-to-end security monitoring.

# Architecture Overview

The solution follows a standard identity federation pattern:

Authentication: User authenticates with Azure AD.

Authorization (Policy Check): Azure Conditional Access enforces MFA.

Assertion: Azure AD generates a signed SAML assertion, including the required https://aws.amazon.com/SAML/Attributes/Role claim.

Assume Role: AWS STS validates the assertion and the Role ARN, granting temporary access credentials.

Access: User lands in the AWS Console with the permissions defined by the assumed IAM role.

# Implementation Details

1. AWS IAM Configuration

The following Trust Policy was applied to the AzureAD_ReadOnlyRole to establish trust with the Identity Provider (IdP) created in AWS.

aws/saml-trust.json

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Principal": {
				"Federated": "arn:aws:iam::************:saml-provider/AzureAD-IdP"
			},
			"Action": "sts:AssumeRoleWithSAML",
			"Condition": {
				"StringEquals": {
					"SAML:aud": "[https://signin.aws.amazon.com/saml](https://signin.aws.amazon.com/saml)"
				}
			}
		}
	]
}


# CLI Snippets (Example)

1. Create the IAM Role with the SAML Trust Policy
aws iam create-role \
  --role-name AzureAD_ReadOnlyRole \
  --assume-role-policy-document file://saml-trust.json

2. Attach the least-privilege policy
aws iam attach-role-policy \
  --role-name AzureAD_ReadOnlyRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess


2. Azure AD Role Claim Workaround

To resolve the Not authorized to perform sts:AssumeRoleWithSAML error caused by Azure AD automatically quoting the role ARN string, the following non-standard, but effective, workaround was implemented:

Claim Name

Configuration

Purpose

https://aws.amazon.com/SAML/Attributes/Role

Source: user.extensionAttribute1

Bypasses quoting by pulling the raw string directly from the user object.

ARN String Value

arn:aws:iam::197**********:role/AzureAD_ReadOnlyRole,arn:aws:iam::197******:saml-provider/AzureAD-IdP

The exact combined string required by AWS STS.

3. Conditional Access Policy

A policy named Require_MFA_for_AWS_Federation was created to enforce MFA.

Setting

Value

Users

Specific Test User (or Groups in a Production environment)

Cloud Apps

AWS-Federation Enterprise Application

Grant

Require multi-factor authentication


