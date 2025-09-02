# Okta Bulk User Deletion Script

**⚠️ IMPORTANT DISCLAIMER:** An Okta user deletion is a permanent process that cannot be reversed. Use these scripts at your own risk. All scripts are provided AS IS without warranty of any kind. Okta disclaims all implied warranties including, without limitation, any implied warranties of fitness for a particular purpose. We highly recommend testing scripts in a preview environment if possible.

## Overview

This PowerShell script (`delete-users.ps1`) enables bulk deletion of Okta users via the Okta API. The script processes users from a CSV file and handles both active and already deprovisioned users appropriately.

## Features

- Bulk delete users from a CSV input file
- Support for both production (`okta.com`) and preview (`oktapreview.com`) environments
- Automatic user deactivation before deletion for active users
- Direct deletion for already deprovisioned users
- Comprehensive logging with separate CSV files for different outcomes
- 1Password CLI integration for secure API key management
- Interactive API key prompt if not provided

## Prerequisites

- PowerShell 6.1 or later
- Valid Okta API token with appropriate permissions
- CSV file containing usernames to delete
- (Optional) 1Password CLI for secure API key management

## Required Okta API Permissions

Your API token must have the following permissions:
- `okta.users.read`
- `okta.users.manage`

## Usage

### Basic Usage

```powershell
.\delete-users.ps1 -orgurl "your-org" -key "your-api-key" -filepath "users.csv"
```

### Using Preview Environment

```powershell
.\delete-users.ps1 -orgurl "your-org" -key "your-api-key" -filepath "users.csv" -preview
```

### Using 1Password CLI

```powershell
.\delete-users.ps1 -orgurl "your-org" -key "op://vault/item/field" -filepath "users.csv"
```

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `-orgurl` | No | `weather` | Your Okta organization subdomain (e.g., 'mycompany' for mycompany.okta.com) |
| `-key` | No* | - | Okta API token. If not provided or too short, you'll be prompted to enter it |
| `-filepath` | Yes | - | Path to CSV file containing users to delete |
| `-preview` | No | - | Switch to use oktapreview.com instead of okta.com |

*The script will prompt for the API key if not provided or if the provided key is too short.

## CSV File Format

The input CSV file must contain a header row with a `login` column:

```csv
login
john.doe@company.com
jane.smith@company.com
user@example.com
```

## Process Flow

For each user in the CSV file, the script:

1. **Retrieves user information** from Okta
2. **Checks user status:**
   - If **DEPROVISIONED**: Deletes the user directly
   - If **ACTIVE**: Deactivates the user first, then deletes
3. **Logs the outcome** to appropriate CSV files

## Output Files

The script creates a `Logs` directory with the following CSV files:

### Successful Operations
- `deprov-users-deleted.csv` - Already deprovisioned users successfully deleted
- `active-users-deprovisioned.csv` - Active users successfully deactivated
- `active-users-deprovisioned-deleted.csv` - Active users successfully deactivated and deleted

### Failed Operations
- `deprov-users-deletion-failed.csv` - Failed to delete already deprovisioned users
- `active-users-deprovisioning-failed.csv` - Failed to deactivate active users
- `active-users-deprovisioned-deletion-failed.csv` - Failed to delete after deactivation

### Status Tracking
- `deprov-users.csv` - Users that were already deprovisioned
- `active-users.csv` - Users that were active
- `not-found-users.csv` - Users not found in the organization

## Examples

### Example 1: Delete users from production environment
```powershell
.\delete-users.ps1 -orgurl "mycompany" -key "00abc123..." -filepath ".\users-to-delete.csv"
```

### Example 2: Delete users from preview environment
```powershell
.\delete-users.ps1 -orgurl "mycompany" -key "00abc123..." -filepath ".\users-to-delete.csv" -preview
```

### Example 3: Using 1Password CLI for API key
```powershell
.\delete-users.ps1 -orgurl "mycompany" -key "op://Private/Okta-API/credential" -filepath ".\users-to-delete.csv"
```

## Error Handling

The script includes comprehensive error handling:
- Invalid or missing API keys trigger interactive prompts
- Network errors are caught and logged
- Users not found in the organization are logged separately
- Failed operations are logged with detailed error information

## Best Practices

1. **Test in Preview First**: Always test with the `-preview` parameter in your preview environment
2. **Backup Data**: Ensure you have backups before running bulk deletions
3. **Review CSV File**: Double-check your CSV file contains only the users you intend to delete
4. **Monitor Logs**: Review all generated log files after execution
5. **Secure API Keys**: Use 1Password CLI or environment variables instead of hardcoding API keys

## Troubleshooting

### Common Issues

**"Error Occurred While Executing Request"**
- Check your API key permissions
- Verify the organization URL is correct
- Ensure network connectivity to Okta

**"User Not Found in Org"**
- Verify usernames in CSV file are correct
- Check if users exist in the specified organization
- Ensure you're targeting the correct environment (production vs preview)

**API Key Issues**
- Ensure API key has sufficient permissions
- Check if API key has expired
- Verify the key is correctly formatted

## Additional Resources

For more detailed instructions, visit: https://support.okta.com/help/Documentation/Knowledge_Article/How-to-Perform-a-Bulk-Delete-of-Okta-Users-With-API
