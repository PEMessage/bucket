---
name: scoop-manifest-creation
description: Create and validate Scoop bucket manifests for GitHub releases. Handles Windows binary extraction, hash verification, and proper manifest structure. Use when user requests creating Scoop manifests for GitHub projects.
license: MIT
metadata:
  author: opencode
  version: "1.0"
compatibility: Requires scoop, git, curl, jq, and access to GitHub API
---

# Scoop Manifest Creation Skill

## Purpose
Create proper Scoop bucket manifests for GitHub releases with correct architecture support, hash verification, and auto-update configuration.

## When to Use
- User asks to create a Scoop manifest for a GitHub project
- Need to package Windows binaries from GitHub releases
- Setting up auto-updating Scoop packages

## Core Principles

### 1. Never Guess Hashes
- **CRITICAL**: Never use placeholder or guessed SHA256 hashes
- If hash is unknown, omit it entirely - Scoop will prompt user on first install
- Extract actual hashes from GitHub API using proper methods

### 2. Validate Before Committing
- Always test manifests with `scoop install ./manifest.json`
- Verify installation works correctly
- Ensure binaries execute properly

### 3. Follow Scoop Conventions
- Use consistent naming (kebab-case for manifest files)
- Include proper architecture support (64bit, 32bit, arm64)
- Configure auto-update with `checkver: "github"`

## Step-by-Step Workflow

### 1. Research GitHub Releases
```bash
# Check if Windows binaries exist
curl -s https://api.github.com/repos/owner/repo/releases/latest | jq '.assets[].name'
```

### 2. Extract Correct Hashes
```bash
# Method 1: Using jq (recommended if available)
curl -s https://api.github.com/repos/owner/repo/releases/latest | \
  jq '.assets[] | select(.name | contains("windows")) | {name: .name, hash: .digest}'

# Method 2: Using grep (fallback without jq)
curl -s https://api.github.com/repos/owner/repo/releases/latest | \
  grep -A 5 -B 5 '"name".*"windows'
# Manually extract hash from the "digest" field in the JSON response
```

### 3. Create Manifest Template
```json
{
    "version": "x.x.x",
    "description": "Tool description",
    "homepage": "https://github.com/owner/repo",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/owner/repo/releases/download/vx.x.x/binary-windows-amd64",
            "hash": "sha256:ACTUAL_HASH_FROM_API"
        },
        "32bit": {
            "url": "https://github.com/owner/repo/releases/download/vx.x.x/binary-windows-386",
            "hash": "sha256:ACTUAL_HASH_FROM_API"
        }
    },
    "bin": "binary.exe",
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/owner/repo/releases/download/v$version/binary-windows-amd64"
            },
            "32bit": {
                "url": "https://github.com/owner/repo/releases/download/v$version/binary-windows-386"
            }
        }
    }
}
```

### 4. Binary Naming
Most Go projects release binaries with `.exe` extension. If binary lacks extension, maintainers should fix release naming rather than using `pre_install` workarounds.

### 5. Test Installation
```bash
# Test the manifest locally
scoop install ./manifest.json

# If hash is missing, Scoop will prompt for verification
# User can accept the calculated hash
```

### 6. Add to Bucket
```bash
# After successful testing
git add manifest.json
git commit -m "Add tool-name vx.x.x"
```

## Common Patterns

### Go Binaries
Most Go projects now include `.exe` extension for Windows binaries. If extension is missing, contact maintainer to fix release naming.

### Multiple Architectures
```json
"architecture": {
    "64bit": { "url": "...-amd64", "hash": "..." },
    "32bit": { "url": "...-386", "hash": "..." },
    "arm64": { "url": "...-arm64", "hash": "..." }
}
```

### Auto-update Configuration
```json
"checkver": "github",
"autoupdate": {
    "architecture": {
        "64bit": { "url": "https://.../download/v$version/binary-amd64" },
        "32bit": { "url": "https://.../download/v$version/binary-386" }
    }
}
```

## Error Handling

### Missing Windows Binaries
If no Windows binaries in release:
1. Check if project builds for Windows
2. Suggest user contact maintainer
3. Consider alternative packaging methods

### Incorrect Hash Format
GitHub API returns `digest` field with format `sha256:HASH`. Use as-is in Scoop manifest (Scoop accepts full `sha256:HASH` format).

### Version Tag Format
Ensure `$version` in autoupdate matches release tag format. Use `checkver: "github"` for automatic version detection.

## Validation Checklist
- [ ] Windows binaries exist in GitHub release
- [ ] SHA256 hashes extracted from GitHub API (not guessed)
- [ ] Manifest tested with `scoop install`
- [ ] Binary executes correctly after installation
- [ ] Auto-update URLs match release pattern
- [ ] Binary has proper `.exe` extension (no renaming needed)

## References
- [Scoop Manifest Format](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests)
- [GitHub Releases API](https://docs.github.com/en/rest/releases/releases)
- [Scoop Auto-update](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests#autoupdate)

## Examples
See `convert-cbz.json` in this bucket for a working example (note: hashes omitted for user verification).