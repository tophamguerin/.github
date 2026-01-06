# Security & Dependency Management

This document outlines the security automation in place across all TG repositories.

## What's Protected

All repositories have automated security tooling:

| Tool | Purpose | Frequency |
|------|---------|-----------|
| **Renovate** | Dependency updates | Weekly PRs (Mondays before 9am) |
| **Semgrep** | Security scanning (SAST) | Every push and PR |
| **Dependabot Alerts** | Vulnerability detection | Continuous |

## How It Works

### Renovate (Dependency Updates)
- Scans `package.json`, `requirements.txt`, `pyproject.toml`, etc.
- Creates batched PRs weekly to reduce noise
- Patch updates auto-merge if CI passes
- Security vulnerabilities get priority labels

### Semgrep (Code Security)
- Runs on every push to main/master
- Runs on every pull request
- Scans for:
  - Security vulnerabilities (OWASP Top 10)
  - Hardcoded secrets
  - SQL injection, XSS, etc.

### Dependabot Alerts
- GitHub's built-in vulnerability scanner
- Alerts appear in the Security tab of each repo
- Critical alerts trigger Slack notifications

## Slack Notifications

Security alerts are sent to `#github-alerts`:
- New vulnerability PRs from Renovate
- Failed security scans
- Dependabot security alerts

---

## Setting Up a New Repository

When creating a new repo, add these files:

### 1. `renovate.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "schedule": ["before 9am on monday"],
  "groupName": "all dependencies",
  "groupSlug": "all",
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true,
      "automergeType": "pr"
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"]
  }
}
```

### 2. `.github/workflows/semgrep.yml`

```yaml
name: Semgrep
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    container:
      image: semgrep/semgrep
    steps:
      - uses: actions/checkout@v4
      - run: semgrep scan --config p/security-audit --config p/secrets --error
```

### 3. Subscribe to Slack

Run in `#github-alerts`:
```
/github subscribe tophamguerin/YOUR-REPO-NAME issues pulls workflows
```

---

## Responding to Alerts

### Severity Levels

| Severity | Response Time | Action |
|----------|---------------|--------|
| Critical | 24 hours | Fix immediately, deploy ASAP |
| High | 1 week | Include in next sprint |
| Medium | 1 month | Prioritize as needed |
| Low | Quarterly | Review during maintenance |

### When a Vulnerability is Found

1. Check the Slack alert or GitHub Security tab
2. Review the Renovate PR or Dependabot alert
3. Test the update locally if needed
4. Merge the fix
5. Deploy

---

## Questions?

Contact the dev team or check the GitHub Security tab on any repo.
