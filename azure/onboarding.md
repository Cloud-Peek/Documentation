# Azure Authentication Guide for CloudPeek

This guide explains how to obtain the necessary Azure credentials for authenticating CloudPeek to scan your Azure resources. This documentation follows Microsoft's best practices for service principal authentication.

## Prerequisites

- An Azure account with at least [Application Administrator](https://learn.microsoft.com/en-us/azure/active-directory/roles/permissions-reference#application-administrator) or [Cloud Application Administrator](https://learn.microsoft.com/en-us/azure/active-directory/roles/permissions-reference#cloud-application-administrator) role to register applications
- For subscription access, at least [User Access Administrator](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#user-access-administrator) to assign roles
- Access to the subscription you want to scan

## Step 1: Create an Application Registration

1. Sign in to the [Azure Portal](https://portal.azure.com)
2. Navigate to **Azure Active Directory** → **App registrations**
3. Click **+ New registration**
4. Enter the following details:
   - **Name**: CloudPeekAuditor (or your preferred name)
   - **Supported account types**: Accounts in this organizational directory only (Single tenant)
   - **Redirect URI**: (Leave blank - not needed for service authentication)
5. Click **Register**

> **Note**: For detailed instructions on creating an application registration, refer to the [Microsoft documentation](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app).

## Step 2: Note Down the Application (Client) ID and Tenant ID

After creating the application, you'll be taken to its overview page:

1. Copy the **Application (client) ID** - This is your `AZURE_CLIENT_ID`
2. Copy the **Directory (tenant) ID** - This is your `AZURE_TENANT_ID`

## Step 3: Create a Client Secret

1. In your application page, navigate to **Certificates & secrets** in the left menu
2. Under **Client secrets**, click **+ New client secret**
3. Enter a description (e.g., "CloudPeek Secret")
4. Select an expiration period (6 months, 1 year, 2 years, or never)
5. Click **Add**
6. **IMPORTANT**: Copy the **Value** of the secret immediately - this is your `AZURE_CLIENT_SECRET`
   - This value will only be shown once and cannot be retrieved later

> **Security Note**: Microsoft recommends setting an expiration for secrets rather than using "never expires". See [Microsoft's security recommendations](https://learn.microsoft.com/en-us/azure/active-directory/develop/security-best-practices-for-app-registration#configure-authentication-and-credential-management).

## Step 4: Assign Read-Only Permissions to Your Subscription

For a read-only auditor role that can scan resources without making changes, you should assign the **Reader** role:

1. Navigate to **Subscriptions** in the Azure Portal 
2. Select the subscription you want to scan
3. Click on **Access control (IAM)** in the left menu
4. Click **+ Add** → **Add role assignment**
5. In the **Role** tab, select the appropriate read-only role:
   - **Reader**: Allows viewing all resources but cannot make changes
   - For more granular audit-only access, consider [Azure built-in roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles) like:
     - **Security Reader**: Can view security-related components, policies, and settings
     - **Resource Policy Contributor**: Can create and modify resource policies but not enforce them
6. In the **Members** tab, select **User, group, or service principal**
7. Click **+ Select members**
8. Search for your application name (e.g., CloudPeekAuditor)
9. Select the application and click **Select**
10. Click **Review + assign** and then **Assign**

> **Note**: For detailed information on Azure RBAC roles, see the [official RBAC documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview).

### Additional Permissions for Specific Resources (Optional)

For scanning specific resources, you may need additional read permissions:

- **Storage Account Reader and Data Access**: To scan storage account data
- **Key Vault Reader**: To audit key vault properties (not secrets)
- **Cosmos DB Reader**: For Cosmos DB auditing

These can be assigned at the resource level following similar steps as above.

## Step 5: Get Your Subscription ID

1. Navigate to **Subscriptions** in the Azure Portal
2. Select the subscription you want to scan
3. Copy the **Subscription ID** - This is your `AZURE_SUBSCRIPTION_ID`

> **Note**: For more information on working with multiple subscriptions, see [Microsoft's guidance on managing multiple subscriptions](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/scale-subscriptions).

## Step 6: Use These Credentials with CloudPeek

To add onbaord your workload please do the folloiwng
* logon to cloudpeek [https://app.cloudpeek.io]
* Go to Settings -> Workloads
* Click on Azure and fill out the form

## Troubleshooting Authentication

### Testing Your Credentials

Before adding your credentials to CloudPeek, test your credentials with the Azure CLI:

```bash
az login --service-principal \
  --username your_client_id \
  --password your_client_secret \
  --tenant your_tenant_id
```

If this succeeds, your credentials are correct.

For additional verification, test resource access:

```bash
# List resource groups to verify read access
az group list --query "[].name" -o tsv
```

### Common Errors

1. **"Application with identifier X was not found in the directory"**
   - Verify your `AZURE_CLIENT_ID` matches the Application ID from App registrations
   - Reference: [Microsoft troubleshooting guide](https://learn.microsoft.com/en-us/azure/active-directory/develop/troubleshoot-service-principal-authentication)

2. **"AADSTS7000215: Invalid client secret provided"**
   - Your client secret is incorrect or expired
   - Create a new client secret and update your script
   - Reference: [Credential management](https://learn.microsoft.com/en-us/azure/active-directory/develop/security-best-practices-for-app-registration#credential-management)

3. **"Insufficient privileges to complete the operation"**
   - The service principal needs additional role assignments on your subscription
   - Reference: [Role assignment troubleshooting](https://learn.microsoft.com/en-us/azure/role-based-access-control/troubleshooting)

## Security Best Practices

- **Rotate secrets regularly**: Create a new client secret periodically and update your configuration
  - Reference: [Microsoft's credential rotation guidance](https://learn.microsoft.com/en-us/azure/active-directory/develop/security-best-practices-for-app-registration#implement-proper-credential-rotation)

- **Use minimum required permissions**: Assign only the roles needed for your scanning
  - For read-only auditing, never assign roles with write permissions
  - Follow the [principle of least privilege](https://learn.microsoft.com/en-us/azure/active-directory/develop/secure-least-privileged-access)

- **Consider Managed Identities**: For CloudPeek running in Azure, use managed identities instead of service principals
  - Reference: [Managed identities documentation](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)

- **Store secrets securely**: Use Azure Key Vault or another secure secret store
  - Reference: [Azure Key Vault security documentation](https://learn.microsoft.com/en-us/azure/key-vault/general/security-features)

- **Implement Conditional Access**: Consider implementing [Conditional Access policies](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/overview) to restrict access based on conditions

- **Monitor Service Principal Activity**: Use [Azure AD sign-in logs](https://learn.microsoft.com/en-us/azure/active-directory/reports-monitoring/concept-sign-ins) to monitor service principal activity

## Recommended Read-Only Roles for CSPM

For Cloud Security Posture Management (CSPM) with read-only access, consider these specific role combinations:

### Core Auditing Roles

| Role | Purpose | Documentation |
|------|---------|--------------|
| **Reader** | Basic read access to all resources | [Reader role](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#reader) |
| **Security Reader** | View security settings and status | [Security Reader role](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#security-reader) |
| **Policy Insights Reader** | View compliance results | [Policy Insights Reader](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#policy-insights-data-reader-preview) |

### Advanced Auditing Roles (Optional)

| Role | Purpose | Documentation |
|------|---------|--------------|
| **Key Vault Reader** | Review key vault configurations | [Key Vault Reader](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#key-vault-reader) |
| **Network Contributor** (read operations only) | Complete network assessments | [Network Contributor](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#network-contributor) |
| **Storage Account Key Operator Service** | Check storage security settings | [Storage roles](https://learn.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json) |

> **Note**: You can create a [custom role](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles) that combines read-only permissions across multiple resource types for a more tailored auditing experience.

## Next Steps

- Set up scheduled runs for regular scanning
- Implement notification systems for compliance violations
- Create compliance dashboards with the scan results
- For comprehensive CSPM, consider Microsoft's built-in tools like [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)

## Additional Resources

- [Microsoft Security Best Practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/identity-management-best-practices)
- [Azure Security Benchmarks](https://learn.microsoft.com/en-us/azure/security/benchmarks/introduction)
