# Using AWS SSO Session Keys for Temporary CloudPeek Access

This document provides instructions for obtaining temporary credentials for CloudPeek integration using AWS Single Sign-On (SSO).

## Overview

These instructions allow you to generate temporary AWS access credentials from an existing AWS SSO session. This approach offers enhanced security by leveraging short-lived credentials rather than permanent access keys.

## Prerequisites

- You must have AWS SSO configured in your organization
- You must have permissions to access the AWS account and appropriate permissions

## Console Instructions using AWS SSO

### Step 1: Sign in to AWS using SSO

1. Access your organization's AWS SSO portal URL (typically something like `https://your-domain.awsapps.com/start`)
2. Enter your credentials and complete any multi-factor authentication if required
3. From the AWS accounts page, find the account you need to configure CloudPeek for
4. Select the appropriate permission set (you'll need one with sufficient permissions, such as SecurityAuditor)
5. Click "Management console" to access the AWS Console

### Step 2: Create a temporary credentials profile

1. Once in the AWS Management Console, click on your username in the top-right corner
2. Select "Security credentials" from the dropdown menu
3. Scroll down to the "Access keys" section
4. Click "Create access key"
5. Select "Command Line Interface (CLI)" as the use case
6. Check the "I understand the above recommendation..." confirmation box
7. Click "Next"
8. (Optional) Add a description tag such as "CloudPeek temporary access"
9. Click "Create access key"
10. You will see your Access key ID and Secret access key. Either:
    - Click "Download .csv file" to save the credentials
    - Copy both the Access key ID and Secret access key to a secure location

### Step 3: Configure CloudPeek with temporary credentials

1. Access your CloudPeek settings "/settings/provider"
2. Navigate to the AWS integration section
3. Enter the temporary credentials:
   - Access Key ID
   - Secret Access Key
   - AWS SESSION TOKEN
4. Set an appropriate reminder to refresh these credentials before they expire (SSO session credentials typically last 1-12 hours depending on your organization's settings)

## Important Notes

1. **Credential Expiration**: These credentials are temporary and will expire when your SSO session ends
2. **Renewal Process**: You'll need to repeat this process when the credentials expire
3. **Automation Consideration**: For long-term integration, consider setting up a dedicated IAM user with appropriate permissions or using AWS Security Token Service (STS) for automated credential rotation

## Troubleshooting

If CloudPeek reports authentication errors:
- Verify that the credentials were copied correctly
- Check if the SSO session has expired (return to Step 1)
- Confirm that the permission set used has the necessary permissions for CloudPeek operations
