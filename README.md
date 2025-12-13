# vault-scanner

Detect secrets in files using your 1Password vault as the source of truth.

## The Problem

Traditional secret scanners use regex patterns to guess what might be a secret. This leads to:
- **False positives**: Random strings flagged as secrets
- **False negatives**: Custom secrets that don't match known patterns slip through
- **No context**: Scanner doesn't know which secrets are actually yours

## The Solution

`vault-scanner` queries your actual 1Password vault and checks if any of your real secrets appear in the scanned files. If a string in your file matches something in your vault, it's definitely a secret.

```
Your File → 1Password Vault Query → Exact Match Detection → Report
```

## Quick Start

```bash
# Install
git clone https://github.com/UncertainMeow/vault-scanner.git
cd vault-scanner
chmod +x vault-scan vault-sanitize

# Sign in to 1Password
eval $(op signin)

# Scan a file
./vault-scan config.yml
```

## Tools

### vault-scan

Detects secrets by cross-referencing files against your 1Password vault.

```bash
# Basic scan
vault-scan document.md

# Scan specific vault
vault-scan -v Infrastructure server-config.yml

# JSON output for CI/CD
vault-scan -f json deployment.yml

# Multiple files
vault-scan *.md *.yml
```

**Exit codes:**
- `0` - No secrets found
- `1` - Secrets detected

### vault-sanitize

Pattern-based sanitizer for redacting common secret formats.

```bash
# Sanitize to new file
vault-sanitize config.yml config-clean.yml

# In-place (creates backup)
vault-sanitize -i sensitive.md

# Strict mode (emails, UUIDs, bearer tokens)
vault-sanitize -s --strict document.md clean.md

# Preserve IP addresses
vault-sanitize -p infrastructure.md safe.md
```

## Workflow

```bash
# 1. Scan file against your vault
vault-scan captured-session.md
# Output: "ALERT: 2 secret(s) found (2 high risk)"

# 2. Sanitize the file
vault-sanitize captured-session.md safe-session.md

# 3. Verify sanitization
vault-scan safe-session.md
# Output: "OK: No secrets detected"

# 4. Safe to share with AI/others
```

## What Gets Detected

### By vault-scan (your actual secrets)
- Passwords from Login items
- API keys matching known prefixes (sk-, ghp_, etc.)
- Tokens and credentials stored in your vault
- URLs that may contain embedded credentials

### By vault-sanitize (patterns)
- **API Keys**: OpenAI, Anthropic, GitHub, GitLab, Slack, AWS, Stripe
- **Passwords**: Common password=value patterns
- **Database URLs**: MySQL, PostgreSQL, MongoDB, Redis connection strings
- **Private Keys**: RSA, EC, and generic private key blocks
- **Network**: IP addresses, MAC addresses (optional)
- **Infrastructure**: Proxmox tokens, CSRF tokens

## Installation

### Prerequisites

```bash
# 1Password CLI
brew install 1password-cli

# jq for JSON processing
brew install jq

# Sign in
eval $(op signin)
```

### Install vault-scanner

```bash
git clone https://github.com/UncertainMeow/vault-scanner.git
cd vault-scanner
chmod +x vault-scan vault-sanitize

# Optional: add to PATH
ln -s "$(pwd)/vault-scan" ~/bin/vault-scan
ln -s "$(pwd)/vault-sanitize" ~/bin/vault-sanitize
```

## CI/CD Integration

```yaml
# GitHub Actions example
- name: Check for secrets
  run: |
    eval $(op signin)
    vault-scan --quiet deployment/*.yml
```

```bash
# Pre-commit hook
#!/bin/bash
vault-scan --quiet $(git diff --cached --name-only) || {
    echo "Secrets detected! Run vault-sanitize first."
    exit 1
}
```

## Custom Patterns

Create a patterns file for organization-specific secrets:

```bash
# my-patterns.txt
s/MYCOMPANY_TOKEN=[A-Za-z0-9]+/MYCOMPANY_TOKEN=[REDACTED]/g
s/internal-api-[a-z0-9]{16}/internal-api-[REDACTED]/g
```

```bash
vault-sanitize -c my-patterns.txt document.md clean.md
```

## How It Works

### vault-scan

1. Queries your 1Password vault via the `op` CLI
2. Extracts secrets (passwords, API keys, tokens) from each item
3. Searches input files for exact matches
4. Reports findings with item names (not the secrets themselves)

### vault-sanitize

1. Applies sed patterns to redact known secret formats
2. Optionally sanitizes IP/MAC addresses
3. Adds sanitization header to output
4. Verifies no obvious secrets remain

## Security Notes

- **Secrets never logged**: vault-scan reports item names, not secret values
- **Temp files cleaned**: All temporary files removed on exit
- **Local processing**: Nothing sent externally
- **Audit trail**: JSON reports for compliance

## Comparison

| Feature | vault-scanner | Regex Scanners | git-secrets |
|---------|--------------|----------------|-------------|
| Finds YOUR secrets | Yes | No | No |
| False negatives | Zero* | Many | Many |
| False positives | Minimal | Many | Moderate |
| Custom secrets | Automatic | Manual patterns | Manual patterns |
| Setup complexity | 1Password | None | Git hooks |

*For secrets stored in your 1Password vault

## Related

- [khoj-utils](https://github.com/UncertainMeow/khoj-utils) - Session capture with Khoj integration
- [1Password CLI](https://developer.1password.com/docs/cli/) - Official CLI documentation
