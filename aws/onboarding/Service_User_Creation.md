# AWS CloudPeek Service User Setup Documentation

This document outlines the process for setting up the required IAM Group, User, and access keys for CloudPeek integration with AWS.

## Overview

The setup includes:
1. Creating an IAM Group with the SecurityAudit policy
2. Creating a service user
3. Adding the user to the group
4. Generating access keys for the service user

## Console Instructions

### Step 1: Create the IAM Group

1. Sign in to the AWS Management Console
2. Navigate to the IAM service (search for "IAM" in the search bar)
3. In the left navigation pane, click on "User groups"
4. Click the "Create group" button
5. Enter "cloudpeek_service_user_group" as the group name
6. In the "Attach permissions policies" section, search for "SecurityAudit" 
7. Check the box next to the SecurityAudit policy
8. Click "Create group" at the bottom of the page

### Step 2: Create the IAM User

1. In the left navigation pane of IAM, click on "Users"
2. Click the "Add users" button
3. Enter "cloudpeek_service_user" as the user name
4. Under "Select AWS access type", check "Access key - Programmatic access"
5. Click "Next: Permissions"
6. Select the "Add user to group" option
7. Check the box next to "cloudpeek_service_user_group"
8. Click "Next: Tags" (add any tags if required by your organization)
9. Click "Next: Review"
10. Verify the information and click "Create user"
11. **IMPORTANT**: On the success page, you will see the Access key ID and Secret access key. Download the .csv file or copy these values immediately as the secret access key will not be shown again.

## AWS CLI

```bash
# Create the IAM Group
aws iam create-group --group-name cloudpeek_service_user_group

# Attach SecurityAudit policy to the group
aws iam attach-group-policy \
  --group-name cloudpeek_service_user_group \
  --policy-arn arn:aws:iam::aws:policy/SecurityAudit

# Create the IAM User
aws iam create-user --user-name cloudpeek_service_user

# Add the user to the group
aws iam add-user-to-group \
  --user-name cloudpeek_service_user \
  --group-name cloudpeek_service_user_group

# Generate access keys for the user
aws iam create-access-key --user-name cloudpeek_service_user
```

## AWS CloudFormation

```yaml
Resources:
  CloudPeekServiceUserGroup:
    Type: AWS::IAM::Group
    Properties:
      GroupName: cloudpeek_service_user_group
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/SecurityAudit

  CloudPeekServiceUser:
    Type: AWS::IAM::User
    Properties:
      UserName: cloudpeek_service_user
      Groups:
        - !Ref CloudPeekServiceUserGroup

  CloudPeekServiceUserAccessKey:
    Type: AWS::IAM::AccessKey
    Properties:
      UserName: !Ref CloudPeekServiceUser

Outputs:
  AccessKeyId:
    Description: The Access Key ID for the CloudPeek service user
    Value: !Ref CloudPeekServiceUserAccessKey
  
  SecretAccessKey:
    Description: The Secret Access Key for the CloudPeek service user
    Value: !GetAtt CloudPeekServiceUserAccessKey.SecretAccessKey
```

## Terraform

```hcl
resource "aws_iam_group" "cloudpeek_service_user_group" {
  name = "cloudpeek_service_user_group"
}

resource "aws_iam_group_policy_attachment" "security_audit_policy" {
  group      = aws_iam_group.cloudpeek_service_user_group.name
  policy_arn = "arn:aws:iam::aws:policy/SecurityAudit"
}

resource "aws_iam_user" "cloudpeek_service_user" {
  name = "cloudpeek_service_user"
}

resource "aws_iam_user_group_membership" "add_user_to_group" {
  user = aws_iam_user.cloudpeek_service_user.name
  groups = [
    aws_iam_group.cloudpeek_service_user_group.name
  ]
}

resource "aws_iam_access_key" "cloudpeek_service_user_key" {
  user = aws_iam_user.cloudpeek_service_user.name
}

output "access_key_id" {
  value = aws_iam_access_key.cloudpeek_service_user_key.id
}

output "secret_access_key" {
  value     = aws_iam_access_key.cloudpeek_service_user_key.secret
  sensitive = true
}
```

## Accessing the Access Keys

### Console
When using the AWS Console, you must download the CSV file or copy the Secret Access Key immediately after user creation, as it cannot be retrieved later.

### CloudFormation
When using CloudFormation, the access keys will be available in the stack outputs.

### Terraform
When using Terraform, the access keys will be available in the terraform outputs.

### AWS CLI
When using the AWS CLI, the access keys will be displayed in the response to the `create-access-key` command. Be sure to save these immediately as the secret access key cannot be retrieved again.

```json
{
    "AccessKey": {
        "UserName": "cloudpeek_service_user",
        "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "Status": "Active",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
        "CreateDate": "2023-05-02T20:47:55Z"
    }
}
```

## Configuring CloudPeek

Once you have the access keys, configure CloudPeek by going to "settings/provider" with:
- Access Key ID
- Secret Access Key

These credentials provide CloudPeek with the necessary permissions to perform security audits on your AWS environment.

## Security Considerations

- Store the access keys securely
- Consider implementing a key rotation policy
- Monitor the activity of the service user
- Apply the principle of least privilege if more granular permissions are needed
