---
name: scoop-manifest-creation
description: Create Scoop manifests for GitHub releases. Handles single executables, hash verification, and auto-update.
license: MIT
metadata:
  author: opencode
  version: "1.1"
---

# Scoop Manifest Creation

Create Scoop manifests for GitHub releases with proper binary naming and auto-update.

## Core Principles
- Never guess hashes - omit if unknown
- Test with `scoop install ./manifest.json`
- Use URL fragments for executables without `.exe` extension

## Workflow

### 1. Check GitHub Releases
```bash
curl -s https://api.github.com/repos/owner/repo/releases/latest | jq '.assets[].name'
```

### 2. Extract Hashes
```bash
curl -s https://api.github.com/repos/owner/repo/releases/latest | \
  jq '.assets[] | select(.name | contains("windows")) | {name: .name, hash: .digest}'
```

### 3. Create Manifest

#### Single Executable (Go projects)
```json
{
    "version": "2.0.1",
    "description": "Tool description",
    "homepage": "https://github.com/owner/repo",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/owner/repo/releases/download/v2.0.1/tool-windows-amd64#/tool.exe",
            "hash": "sha256:fff1550cce1b5a707941c8f8ed21aa5bca14ba8b31cf04e05ce93a62b5c4a363"
        },
        "32bit": {
            "url": "https://github.com/owner/repo/releases/download/v2.0.1/tool-windows-386#/tool.exe",
            "hash": "sha256:138cbf80af769d459dc97e1ea8069d62a59294b8d5a1e67af7270aa0ea5ade7a"
        }
    },
    "bin": "tool.exe",
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/owner/repo/releases/download/v$version/tool-windows-amd64#/tool.exe"
            },
            "32bit": {
                "url": "https://github.com/owner/repo/releases/download/v$version/tool-windows-386#/tool.exe"
            }
        }
    }
}
```

### 4. URL Fragments for Binary Naming
Use `#/binary.exe` fragment when GitHub releases lack `.exe` extension:
- `https://.../binary-windows-amd64#/binary.exe`
- Scoop downloads as `binary-windows-amd64`, saves as `binary.exe`
- Apply to all architecture URLs and autoupdate URLs

### 5. Test Installation
```bash
scoop install ./manifest.json
```

## Validation
- [ ] Windows binaries exist
- [ ] Hashes from GitHub API (not guessed)
- [ ] Tested with `scoop install`
- [ ] Binary executes correctly
- [ ] URL fragments used if no `.exe` extension

## Examples
See `convert-cbz.json` in this bucket.