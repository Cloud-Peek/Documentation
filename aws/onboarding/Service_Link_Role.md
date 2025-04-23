# CloudPeek Service Link Role Creation Documentation with Cross-Account Trust and External ID

This document outlines the process for creating a service-linked role for CloudPeek with the SecurityAudit policy attached and establishing a trust relationship with an IAM role in a different AWS account, secured with an external ID condition.

## Service-Linked Role Overview

A service-linked role for CloudPeek will allow the service to perform security audit functions across your AWS environment by leveraging the AWS SecurityAudit managed policy, while also enabling secure cross-account access with an external ID.

## Prerequisites

The CloudPeek ARN (external account) and External ID can be found in your CloudPeek account under the "settings/Providers" section. You will need these values to properly configure the trust relationship.

## Trust Policy

The trust policy defines which entities can assume the role - both the CloudPeek service and a specific IAM role from a different account, with the external account requiring an external ID for additional security:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudpeek.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::EXTERNAL_ACCOUNT_ID:role/EXTERNAL_ROLE_NAME"
      },
      "Action": "sts:AssumeRole",
      "Condition": {"StringEquals": {"sts:ExternalId": "12345"}}
    }
  ]
}
```

> **Note:** Replace `EXTERNAL_ACCOUNT_ID`, `EXTERNAL_ROLE_NAME`, and the External ID (`12345`) with the values from your CloudPeek account's "Add Providers" section.

## Role Creation with AWS CLI

```bash
# Step 1: Create the trust policy JSON file (replace with your external account ID and role name)
cat > cloudpeek-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudpeek.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::EXTERNAL_ACCOUNT_ID:role/EXTERNAL_ROLE_NAME"
      },
      "Action": "sts:AssumeRole",
      "Condition": {"StringEquals": {"sts:ExternalId": "12345"}}
    }
  ]
}
EOF

# Step 2: Create the role with the trust policy
aws iam create-role \
  --role-name CloudPeekServiceRole \
  --assume-role-policy-document file://cloudpeek-trust-policy.json

# Step 3: Attach the SecurityAudit managed policy
aws iam attach-role-policy \
  --role-name CloudPeekServiceRole \
  --policy-arn arn:aws:iam::aws:policy/SecurityAudit
```

## Role Creation with AWS CloudFormation

```yaml
Resources:
  CloudPeekServiceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: CloudPeekServiceRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: cloudpeek.amazonaws.com
            Action: sts:AssumeRole
          - Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${ExternalAccountId}:role/${ExternalRoleName}'
            Action: sts:AssumeRole
            Condition:
              StringEquals:
                'sts:ExternalId': !Ref ExternalId
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/SecurityAudit
    
Parameters:
  ExternalAccountId:
    Type: String
    Description: The CloudPeek AWS Account ID (found in CloudPeek "Add Providers" section)
  
  ExternalRoleName:
    Type: String
    Description: The name of the CloudPeek role (found in CloudPeek "Add Providers" section)
    
  ExternalId:
    Type: String
    Description: The External ID for secure cross-account access (found in CloudPeek "Add Providers" section)
```

## Role Creation with Terraform

```hcl
variable "external_account_id" {
  description = "The CloudPeek AWS Account ID (found in CloudPeek \"Add Providers\" section)"
  type        = string
}

variable "external_role_name" {
  description = "The name of the CloudPeek role (found in CloudPeek \"Add Providers\" section)"
  type        = string
}

variable "external_id" {
  description = "The External ID for secure cross-account access (found in CloudPeek \"Add Providers\" section)"
  type        = string
  default     = "12345"  # Replace with actual External ID from CloudPeek
}

resource "aws_iam_role" "cloudpeek_service_role" {
  name = "CloudPeekServiceRole"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "cloudpeek.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      },
      {
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${var.external_account_id}:role/${var.external_role_name}"
        }
        Action = "sts:AssumeRole"
        Condition = {
          StringEquals = {
            "sts:ExternalId" = var.external_id
          }
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "cloudpeek_security_audit" {
  role       = aws_iam_role_policy_attachment.cloudpeek_service_role.name
  policy_arn = "arn:aws:iam::aws:policy/SecurityAudit"
}
```

## Verification

Verify the role has been created correctly:

```bash
# Verify the role exists
aws iam get-role --role-name CloudPeekServiceRole

# Verify the SecurityAudit policy is attached
aws iam list-attached-role-policies --role-name CloudPeekServiceRole

# Verify the trust policy includes the external account with condition
aws iam get-role --role-name CloudPeekServiceRole --query 'Role.AssumeRolePolicyDocument'
```

## Cross-Account Access Usage

When assuming this role from the external account, the external ID must be provided:

```bash
# Example of assuming the role from the external account
aws sts assume-role \
  --role-arn arn:aws:iam::TARGET_ACCOUNT_ID:role/CloudPeekServiceRole \
  --role-session-name CloudPeekAuditSession \
  --external-id 12345  # Use the External ID from CloudPeek "Add Providers" section
```

## Finding CloudPeek Account Information

1. Log in to your CloudPeek account
2. Navigate to the "setting\Providers" section
3. Look for:
   - CloudPeek AWS Account ID
   - CloudPeek Role Name
   - External ID

Use these exact values in your trust policy configuration to ensure CloudPeek can properly access your account for security auditing.
