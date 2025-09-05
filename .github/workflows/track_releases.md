# Release Tracking Workflow Setup

This document explains how to set up the GitHub workflow that automatically tracks releases to Google Sheets when tags with prefix `rc_*` are created.

## Workflow Overview

The workflow (`track_releases.yaml`) triggers when:
- A new tag is pushed with prefix `rc_*` (e.g., `rc_1.0.0`, `rc_2024.09.05`)

It then stores the following information in Google Sheets:
- **Date**: Current timestamp in UTC
- **Repository**: Full repository name
- **Release Tag**: The tag name that triggered the workflow

## Required Setup

### 1. Google Service Account Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Sheets API:
   - Go to "APIs & Services" > "Library"
   - Search for "Google Sheets API" and enable it
4. Create a service account:
   - Go to "APIs & Services" > "Credentials"
   - Click "Create Credentials" > "Service Account"
   - Fill in the details and create
5. Generate a key for the service account:
   - Click on the created service account
   - Go to "Keys" tab
   - Click "Add Key" > "Create New Key"
   - Choose JSON format and download

### 2. Google Sheets Setup

1. Create a new Google Sheet or use an existing one
2. Add headers in the first row (optional but recommended):
   - Column A: "Date"
   - Column B: "Repository" 
   - Column C: "Release Tag"
3. Share the sheet with your service account email:
   - Click "Share" in Google Sheets
   - Add the service account email (found in the downloaded JSON)
   - Give it "Editor" permissions
4. Copy the Sheet ID from the URL:
   - URL format: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit`

### 3. GitHub Repository Secrets

Add the following secrets in your GitHub repository settings:

#### Secrets (Settings > Secrets and variables > Actions > Secrets)

1. **GOOGLE_SERVICE_ACCOUNT_KEY**
   - Value: The entire content of the downloaded JSON file from step 1.5

2. **CODE_DEPLOYMENT_TRACKING_GOOGLE_SHEET_ID**
   - Value: The sheet ID from step 2.4

#### Variables (Settings > Secrets and variables > Actions > Variables)

1. **GOOGLE_SHEET_RANGE** (optional)
   - Value: `Sheet1!A:C` (default range)
   - Adjust if using different sheet name or columns

## Usage

Once set up, the workflow will automatically run when you push a tag with `rc_*` prefix:

```bash
# Create and push a release candidate tag
git tag rc_20240905
git push origin rc_20240905
```

The workflow will:
1. Extract the tag name and current date
2. Connect to Google Sheets using the service account
3. Append a new row with the release information
4. Log success/failure

## Troubleshooting

### Common Issues

1. **Authentication Error**
   - Verify the service account JSON is correctly set in secrets
   - Ensure the Google Sheets API is enabled in your Google Cloud project

2. **Permission Denied**
   - Make sure the service account email has editor access to the Google Sheet
   - Check that the sheet ID is correct

3. **Sheet Not Found**
   - Verify the Google Sheet ID in the repository secrets
   - Ensure the sheet exists and is accessible

4. **Range Error**
   - Check if the GOOGLE_SHEET_RANGE variable matches your sheet structure
   - Default is `Sheet1!A:C` for columns A, B, C in Sheet1

### Monitoring

- Check the Actions tab in your GitHub repository to see workflow runs
- Each run will show logs including success/failure messages
- The Google Sheet should show new entries after successful runs

## Customization

You can customize the workflow by:

1. **Changing the trigger pattern**: Modify the `tags` filter in the workflow
2. **Adding more data**: Extend the script to include additional information
3. **Different sheet format**: Adjust the range and data structure
4. **Notifications**: Add Slack or email notifications on success/failure

## Security Notes

- The service account key should be kept secure and only have necessary permissions
- Consider using short-lived tokens or workload identity federation for enhanced security
- Regularly rotate service account keys as per security best practices
