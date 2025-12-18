# Load JIRA Issues GitHub Action

Extract JIRA issue references from Git commits for use in downstream actions.

## Features

- **JIRA Cloud & Data Center Support**: Works with both deployment types
- **Automatic Project Key Detection**: Fetches project keys from JIRA API
- **Flexible Changeset Specification**: 8 different changeset types supported
- **JSON Artifact Output**: Structured output for advanced use cases
- **Simple Outputs**: Comma-separated links and keys for easy integration

## Quick Start

### 1. Set Up JIRA API Token

**For JIRA Cloud:**
1. Go to [Atlassian API Tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Create a new API token
3. Add it to GitHub as `JIRA_API_TOKEN` secret

**For JIRA Data Center:**
1. Create a Personal Access Token in JIRA
2. Add it to GitHub as `JIRA_API_TOKEN` secret

### 2. Add to Your Workflow

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Full history for tag/commit ranges

- name: Load JIRA Issues
  id: jira
  uses: sandgardenhq/load-jira-issues@v1
  with:
    jira-base-url: https://company.atlassian.net
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    jira-user-email: user@company.com  # Required for JIRA Cloud
    commits-count: 50

- name: Use JIRA Links
  run: echo "Found issues: ${{ steps.jira.outputs.issue-keys }}"
```

## Inputs

### Required

| Input | Description |
|-------|-------------|
| `jira-base-url` | JIRA instance URL (e.g., `https://company.atlassian.net`) |
| `jira-api-token` | JIRA API token (store in GitHub secrets) |

### Conditional

| Input | Description |
|-------|-------------|
| `jira-user-email` | Required for JIRA Cloud (*.atlassian.net), ignored for Data Center |

### Optional

| Input | Description | Default |
|-------|-------------|---------|
| `project-keys` | Comma-separated JIRA project keys to filter | (all projects) |
| `output-file` | JSON artifact path | `jira-issues.json` |

### Changeset Specification (one required)

Specify changes using **only one** of the following types:

| Input | Description |
|-------|-------------|
| `releases-count` | Number of recent releases |
| `time-range-start` / `time-range-end` | ISO 8601 timestamps |
| `commits-count` | Number of recent commits |
| `commits-since-sha` | Start from specific commit |
| `commits-shas` | Comma-separated specific SHAs |
| `commits-start-sha` / `commits-end-sha` / `commits-include-start` | Commit range |
| `tags-start` / `tags-end` | Tag range |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `issue-links` | Comma-separated browse URLs | `https://company.atlassian.net/browse/PROJ-123,...` |
| `issue-keys` | Comma-separated issue keys | `PROJ-123,PROJ-456,INFRA-789` |
| `issue-count` | Number of unique issues | `5` |

## JSON Artifact

The action writes a JSON file (default: `jira-issues.json`) with detailed data:

```json
{
  "metadata": {
    "generatedAt": "2025-01-15T10:30:00.000Z",
    "jiraBaseUrl": "https://company.atlassian.net",
    "repository": "owner/repo",
    "changeset": { "type": "commits-count", "count": 50 },
    "totalIssues": 5,
    "totalCommits": 50
  },
  "issues": [
    {
      "key": "PROJ-123",
      "url": "https://company.atlassian.net/browse/PROJ-123",
      "commits": ["abc123", "def456"]
    }
  ]
}
```

## Usage Examples

### Extract from Recent Commits

```yaml
- uses: sandgardenhq/load-jira-issues@v1
  with:
    jira-base-url: https://company.atlassian.net
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    jira-user-email: user@company.com
    commits-count: 100
```

### Filter by Project Keys

```yaml
- uses: sandgardenhq/load-jira-issues@v1
  with:
    jira-base-url: https://company.atlassian.net
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    jira-user-email: user@company.com
    commits-count: 50
    project-keys: PROJ,INFRA,OPS
```

### Extract from Tag Range

```yaml
- uses: sandgardenhq/load-jira-issues@v1
  with:
    jira-base-url: https://company.atlassian.net
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    jira-user-email: user@company.com
    tags-start: v1.0.0
    tags-end: v1.1.0
```

### Use with doc-holiday-action

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- name: Load JIRA Issues
  id: jira
  uses: sandgardenhq/load-jira-issues@v1
  with:
    jira-base-url: https://company.atlassian.net
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    jira-user-email: user@company.com
    commits-count: 50

- name: Generate Docs
  uses: sandgardenhq/doc-holiday-action@v1
  with:
    api-token: ${{ secrets.DOC_HOLIDAY_TOKEN }}
    event-type: release
    relevant-links: ${{ steps.jira.outputs.issue-links }}
```

## Prerequisites

- `actions/checkout` must run before this action
- `fetch-depth: 0` recommended for tag/commit range queries

## Development

### Setup

```bash
npm install
```

### Build

```bash
npm run build
```

### Test

```bash
npm test
```

### Package

```bash
npm run package
```

## License

MIT License - see LICENSE file for details.

## Support

- [GitHub Issues](https://github.com/sandgardenhq/load-jira-issues/issues)
