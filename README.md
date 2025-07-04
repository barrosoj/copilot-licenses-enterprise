# GitHub Copilot License Retrieval Action

A reusable GitHub Action to retrieve and report on GitHub Copilot licenses across an enterprise. This action provides comprehensive license management and compliance reporting capabilities.

## Features

- ✅ Retrieve all Copilot seat assignments for an enterprise
- ✅ Support for pagination to handle large enterprises  
- ✅ Multiple output formats (JSON, CSV)
- ✅ Detailed summary reporting with step summaries
- ✅ Both Node.js and Python runtime support
- ✅ Billing summary information
- ✅ Comprehensive error handling

## Usage

### Basic Usage

```yaml
- name: Generate Copilot License Report
  uses: your-org/copilot-licenses-enterprise@v1
  with:
    enterprise: 'your-enterprise-name'
    token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Usage

```yaml
- name: Generate Comprehensive Copilot Report
  id: copilot-report
  uses: your-org/copilot-licenses-enterprise@v1
  with:
    enterprise: 'your-enterprise-name'
    token: ${{ secrets.COPILOT_TOKEN }}
    output-format: 'both'  # Generates both JSON and CSV
    include-billing: true
    include-summary: true
    runtime: 'nodejs'
    output-file: 'monthly_copilot_report'
    per-page: 100

- name: Use the outputs
  run: |
    echo "Total seats: ${{ steps.copilot-report.outputs.total-seats }}"
    echo "JSON file: ${{ steps.copilot-report.outputs.output-file-json }}"
    echo "CSV file: ${{ steps.copilot-report.outputs.output-file-csv }}"
    echo "Organizations: ${{ steps.copilot-report.outputs.organizations }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enterprise` | GitHub Enterprise name | ✅ Yes | - |
| `token` | GitHub token with copilot billing permissions | ✅ Yes | - |
| `output-format` | Output format (`json`, `csv`, or `both`) | ❌ No | `json` |
| `output-file` | Output file name (without extension) | ❌ No | `copilot_licenses` |
| `include-billing` | Include billing summary information | ❌ No | `false` |
| `include-summary` | Show summary information in step summary | ❌ No | `true` |
| `runtime` | Runtime to use (`nodejs` or `python`) | ❌ No | `nodejs` |
| `per-page` | Results per page (max 100) | ❌ No | `100` |

## Outputs

| Output | Description |
|--------|-------------|
| `total-seats` | Total number of Copilot seats |
| `output-file-json` | Path to the generated JSON file |
| `output-file-csv` | Path to the generated CSV file (if CSV format requested) |
| `organizations` | Comma-separated list of organizations with Copilot licenses |

## Required Permissions

Your GitHub token needs the following scopes:
- For enterprise access: `manage_billing:copilot`
- For organization access: `copilot`

## Output Formats

### JSON Format
The JSON output includes detailed information about each seat, billing summary (if requested), and metadata.

### CSV Format
The CSV format is flattened for easy analysis and includes:
- `assignee_login` - Username of the seat assignee
- `organization_login` - Organization name
- `assigning_team_name` - Team name (if applicable)
- `created_at`, `updated_at` - Timestamps
- `last_activity_at`, `last_activity_editor` - Activity information
- `pending_cancellation_date` - If seat is being cancelled

## Example Workflows

### 1. Weekly Scheduled Report

```yaml
name: Weekly Copilot License Report

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM UTC

jobs:
  weekly-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Generate Weekly Report
        uses: your-org/copilot-licenses-enterprise@v1
        with:
          enterprise: ${{ vars.ENTERPRISE_NAME }}
          token: ${{ secrets.COPILOT_TOKEN }}
          output-format: 'both'
          include-billing: true
          output-file: 'weekly_report'
          
      - name: Upload Reports
        uses: actions/upload-artifact@v3
        with:
          name: weekly-copilot-reports
          path: |
            weekly_report_*.json
            weekly_report_*.csv
          retention-days: 90
```

### 2. On-Demand Report with Notifications

```yaml
name: Copilot License Report with Slack

on:
  workflow_dispatch:
    inputs:
      enterprise:
        description: 'Enterprise name'
        required: true

jobs:
  report-and-notify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Generate License Report
        id: report
        uses: your-org/copilot-licenses-enterprise@v1
        with:
          enterprise: ${{ github.event.inputs.enterprise }}
          token: ${{ secrets.COPILOT_TOKEN }}
          include-billing: true
          include-summary: false
          
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: success
          text: |
            📊 Copilot Report Generated
            💺 Total Seats: ${{ steps.report.outputs.total-seats }}
            🏢 Organizations: ${{ steps.report.outputs.organizations }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 3. Compliance Report with Email

```yaml
name: Monthly Compliance Report

on:
  schedule:
    - cron: '0 8 1 * *'  # First day of month at 8 AM UTC

jobs:
  compliance-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Generate Compliance Report
        id: compliance
        uses: your-org/copilot-licenses-enterprise@v1
        with:
          enterprise: ${{ vars.ENTERPRISE_NAME }}
          token: ${{ secrets.COPILOT_TOKEN }}
          output-format: 'csv'
          include-billing: true
          output-file: 'compliance_report'
          
      - name: Email Report
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.company.com
          server_port: 587
          username: ${{ secrets.SMTP_USERNAME }}
          password: ${{ secrets.SMTP_PASSWORD }}
          subject: "Monthly Copilot Compliance Report"
          to: compliance@company.com
          body: |
            Monthly GitHub Copilot license compliance report.
            
            Total Seats: ${{ steps.compliance.outputs.total-seats }}
            Report Date: $(date)
          attachments: ${{ steps.compliance.outputs.output-file-csv }}
```

## Error Handling

The action includes comprehensive error handling for:
- Invalid or expired tokens
- Network connectivity issues
- API rate limiting
- Invalid enterprise names
- Malformed responses

## Security Considerations

1. **Token Storage**: Always store your GitHub token in repository secrets, never in plain text
2. **Permissions**: Use tokens with minimal required permissions
3. **Artifact Retention**: Set appropriate retention periods for generated reports
4. **Access Control**: Limit who can run workflows that generate license reports

## API Reference

This action uses the GitHub Enterprise API endpoint:
```
GET https://api.github.com/enterprises/{enterprise}/copilot/billing/seats
```

For more information, see the [GitHub API documentation](https://docs.github.com/en/rest/copilot/copilot-user-management).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details.
