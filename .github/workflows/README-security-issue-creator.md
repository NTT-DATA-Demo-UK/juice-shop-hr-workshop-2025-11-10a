# Security Issue Creator Workflow

## Overview

The `security-issue-creator.yml` workflow automatically creates GitHub issues for security findings discovered by GitHub's security scanning tools (CodeQL and Secret Scanning).

## Features

- **Automatic Issue Creation**: Creates a GitHub issue for each open security alert
- **Deduplication**: Prevents duplicate issues by checking existing security issues
- **Copilot Assignment**: Attempts to assign issues to `@copilot` for automated resolution
- **Comprehensive Details**: Each issue includes:
  - Alert severity level
  - Detailed description and help text
  - File location and line number
  - Direct link to the security alert
  - Remediation guidance

## Triggers

The workflow runs in three scenarios:

1. **After CodeQL Scan**: Automatically triggers when the "CodeQL Scan" workflow completes successfully
2. **Scheduled**: Runs daily at 2:00 AM UTC to check for new security alerts
3. **Manual**: Can be triggered manually via workflow_dispatch from the Actions tab

## Permissions Required

The workflow requires the following permissions:
- `contents: read` - To check out the repository
- `issues: write` - To create issues
- `security-events: read` - To access security scanning alerts

## Issue Labels

Created issues are automatically labeled with:
- `security-finding` - Identifies issues created from security scans
- `automated` - Indicates the issue was created automatically
- Severity level: `critical`, `high`, `medium`, or `low`
- `secret-exposed` - Additional label for secret scanning findings

## How It Works

1. **Fetch Alerts**: The workflow queries GitHub's API for open code scanning and secret scanning alerts
2. **Check for Duplicates**: Compares alert information with existing issues to avoid creating duplicates
3. **Create Issues**: For each new alert, creates a GitHub issue with:
   - Descriptive title including severity and alert number
   - Detailed body with location, description, and remediation steps
   - Appropriate labels
   - Assignment to @copilot (if available)
4. **Rate Limiting**: Implements 1-second delays between issue creations to respect API limits

## Issue Format

### Code Scanning Issues

```
Title: [Security] medium: SQL Injection vulnerability (CodeQL Alert #123)

Body:
- Alert ID and rule information
- Severity level
- Description and help text
- File location and line number
- Link to the alert
- Automatic note about the issue source
```

### Secret Scanning Issues

```
Title: [Security] Secret detected: GitHub Personal Access Token (Secret Alert #456)

Body:
- Alert ID and secret type
- State and bypass information
- Location details
- Recommended actions for rotation and remediation
- Link to the alert
- Critical security warning
```

## Troubleshooting

### Issues Not Being Created

If the workflow runs but doesn't create issues:
1. Check that security scanning is enabled for the repository
2. Verify the workflow has the necessary permissions
3. Check the workflow logs for API errors
4. Ensure there are open security alerts in the Security tab

### Copilot Not Being Assigned

If issues are created without the copilot assignee:
- This is expected behavior if copilot is not available as a collaborator
- Issues will still be created with proper labels and can be manually assigned

### Rate Limiting

The workflow includes built-in rate limiting (1 second between requests). If you encounter rate limit errors:
- The workflow will fail gracefully
- Re-run the workflow after the rate limit resets
- Consider reducing the number of alerts by addressing existing ones

## Manual Execution

To manually trigger the workflow:

1. Go to the Actions tab in GitHub
2. Select "Create Security Issues from Scan Results"
3. Click "Run workflow"
4. Select the branch and click "Run workflow"

## Integration with Other Workflows

This workflow is designed to work alongside:
- `codeql-analysis.yml` - Triggers after CodeQL scans
- `zap_scan.yml` - Can be extended to process ZAP scan results
- Other security scanning workflows

## Customization

To modify the workflow behavior, edit `.github/workflows/security-issue-creator.yml`:

- **Change schedule**: Modify the `cron` expression in the `schedule` trigger
- **Adjust labels**: Edit the `labels` array in the `createIssue` function calls
- **Modify issue format**: Update the title and body templates
- **Change assignee**: Modify the `assignees` array (default: `['copilot']`)

## Security Considerations

- The workflow uses the `GITHUB_TOKEN` provided by Actions
- API requests are made over HTTPS
- Sensitive information in alerts is handled according to GitHub's security practices
- The workflow only reads security events and creates issues; it does not modify code
