# GitHub Copilot License Retrieval Action

A lightweight, reusable GitHub Action to retrieve and report on GitHub Copilot licenses across an enterprise. Perfect for license management, compliance reporting, and billing analysis.

## 🚀 Features

- ✅ **Multiple output formats** - JSON and CSV with clean, essential data
- ✅ **Pagination support** - Handles enterprises with hundreds of seats
- ✅ **Artifact integration** - Works seamlessly with `upload-artifact`
- ✅ **Comprehensive outputs** - Total seats, file paths, and organization data

## 📊 Output Data

### JSON Format
```json
{
  "total_seats": 150,
  "retrieved_at": "2025-07-04T12:00:00Z",
  "seats": [
    {
      "assignee": {
        "login": "username"
      },
      "pending_cancellation_date": null,
      "plan_type": "business",
      "last_activity_at": "2025-07-03T17:51:55Z",
      "last_activity_editor": "vscode/1.101.2",
      "assigning_team": {
        "name": "development-team"
      },
      "organization": {
        "login": "my-org"
      }
    }
  ]
}
```

### CSV Format
Clean, spreadsheet-friendly columns:
- `assignee_login` - Username
- `pending_cancellation_date` - Cancellation date (if any)
- `plan_type` - Copilot plan type
- `last_activity_at` - Last activity timestamp
- `last_activity_editor` - Editor used
- `assigning_team_name` - Team that assigned the license
- `organization_login` - Organization name

## 🔧 Usage

### Basic Usage

```yaml
- name: Generate Copilot License Report
  uses: your-org/copilot-licenses-enterprise@v1
  with:
    enterprise: 'your-enterprise-name'
    token: ${{ secrets.COPILOT_TOKEN }}
```

### Advanced Usage with Artifacts

```yaml
- name: Generate Copilot License Report
  id: copilot-report
  uses: your-org/copilot-licenses-enterprise@v1
  with:
    enterprise: 'your-enterprise-name'
    token: ${{ secrets.COPILOT_TOKEN }}
    output-format: 'both'  # Generates both JSON and CSV
    include-billing: true
    include-summary: true
    output-file: 'monthly_copilot_report'

- name: Upload Reports as Artifacts
  uses: actions/upload-artifact@v4
  with:
    name: copilot-license-reports
    path: |
      ${{ steps.copilot-report.outputs.output-file-json }}
      ${{ steps.copilot-report.outputs.output-file-csv }}
    retention-days: 90

- name: Display Summary
  run: |
    echo "📊 Total Copilot Seats: ${{ steps.copilot-report.outputs.total-seats }}"
    echo "🏢 Organizations: ${{ steps.copilot-report.outputs.organizations }}"
```

## 📝 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enterprise` | GitHub enterprise name | ✅ Yes | - |
| `token` | GitHub token with `manage_billing:copilot` scope | ✅ Yes | - |
| `output-format` | Output format: `json`, `csv`, or `both` | ❌ No | `json` |
| `include-billing` | Include billing summary information | ❌ No | `false` |
| `include-summary` | Show summary in step output | ❌ No | `true` |
| `output-file` | Base name for output files | ❌ No | `copilot_licenses` |
| `per-page` | Results per API page (max 100) | ❌ No | `100` |

## 📤 Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `total-seats` | Total number of Copilot seats | `247` |
| `output-file-json` | Path to generated JSON file | `copilot_licenses.json` |
| `output-file-csv` | Path to generated CSV file | `copilot_licenses.csv` |
| `organizations` | Comma-separated list of organizations | `org1,org2,org3` |

## 🔒 Required Token Permissions

Your GitHub Personal Access Token needs:

- **Enterprise**: `manage_billing:copilot` scope
- **Organization**: `copilot` scope

## 📋 Example Workflow

```yaml
name: Copilot License Report
on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday at 9 AM

jobs:
  license-report:
    runs-on: ubuntu-latest
    steps:
    - name: Generate License Report
      id: report
      uses: your-org/copilot-licenses-enterprise@v1
      with:
        enterprise: 'my-company'
        token: ${{ secrets.COPILOT_TOKEN }}
        output-format: 'both'
        
    - name: Upload Reports
      uses: actions/upload-artifact@v4
      with:
        name: copilot-reports-${{ github.run_number }}
        path: |
          ${{ steps.report.outputs.output-file-json }}
          ${{ steps.report.outputs.output-file-csv }}
        retention-days: 90
```

## 🛠️ Local Development

You can also run the script locally for testing:

```bash
# Set environment variables
export GITHUB_ENTERPRISE="your-enterprise"
export GITHUB_TOKEN="ghp_your_token"

# Run the script
node get_copilot_licenses.js --output licenses.csv --format csv
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

```http
GET https://api.github.com/enterprises/{enterprise}/copilot/billing/seats
```

For more information, see the [GitHub API documentation](https://docs.github.com/en/rest/copilot/copilot-user-management).

## License

MIT License - see [LICENSE](LICENSE) file for details.
