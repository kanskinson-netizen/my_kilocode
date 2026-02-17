# Upstream Merge Guide

**Purpose**: Guide for merging upstream Roo Code changes into Kilo Code  
**Last Updated**: 2026-02-17

---

## Overview

Kilo Code is a fork of [Roo Code](https://github.com/RooVetGit/Roo-Code). We periodically merge upstream changes to stay up-to-date with bug fixes and new features.

---

## Merge Strategy

### UI Changes (webview-ui/)

**Rule**: Preserve Kilo Code UI customizations

- Accept upstream changes first
- Reapply UI modifications using patch file
- Test UI functionality after merge

### Other Code

**Rule**: Standard merge with kilocode_change markers

- Preserve all `kilocode_change` marked sections
- Integrate upstream changes around marked sections
- Run tests and type checking

---

## Pre-Merge Checklist

- [ ] Commit all local changes
- [ ] Create backup branch: `git checkout -b backup-before-merge-$(date +%Y%m%d)`
- [ ] Ensure tests pass: `pnpm test`
- [ ] Ensure type checking passes: `pnpm check-types`
- [ ] Note current commit hash: `git rev-parse HEAD`

---

## Merge Process

### Step 1: Fetch Upstream

```bash
git remote add upstream https://github.com/RooVetGit/Roo-Code.git
git fetch upstream
git tag -l | grep -E "v[0-9]+\.[0-9]+\.[0-9]+" | sort -V | tail -10
```

### Step 2: Start Merge

```bash
TARGET_RELEASE="v1.2.3"  # Replace with actual release tag
git checkout -b merge-upstream-${TARGET_RELEASE}
git merge ${TARGET_RELEASE}
```

---

## Conflict Resolution

### UI Conflicts (webview-ui/)

**Option A: Apply Patch File (Recommended)**

```bash
# After completing the merge
git apply .kilocode/patches/ui-changes.patch

# If patch applies cleanly
git add webview-ui/
git commit -m "reapply: UI modifications from patch"

# If patch fails (conflicts)
git apply --reject .kilocode/patches/ui-changes.patch
# Manually apply failed changes from .rej files
git add webview-ui/
git commit -m "reapply: UI modifications (manual resolution)"
```

**Option B: Manual Resolution**

1. Keep Kilo Code UI styling, layout, and branding
2. Integrate upstream functional changes (new features, bug fixes)
3. Test UI: `cd webview-ui && pnpm dev`

**Checklist:**

- [ ] Kilo Code branding preserved
- [ ] Upstream functional changes integrated
- [ ] UI functionality tested

### Other Code Conflicts

**For files with kilocode_change markers:**

```typescript
// Example resolution
import { upstreamFeature } from "./upstream-feature"
// kilocode_change start
import { kiloCodeFeature } from "./kilocode-feature"
// kilocode_change end

function processData(data: Data) {
	const validated = upstreamFeature.validate(data)
	// kilocode_change start
	const enhanced = kiloCodeFeature.enhance(validated)
	return enhanced
	// kilocode_change end
}
```

**Checklist:**

- [ ] All kilocode_change sections preserved
- [ ] Upstream functionality integrated
- [ ] Tests pass for affected modules

---

## Post-Merge Validation

```bash
# Run tests
pnpm test

# Type checking
pnpm check-types

# Build
pnpm build

# Verify .vsix created
ls -lh bin/*.vsix
```

**Manual Testing:**

- [ ] Install extension in VS Code
- [ ] Test basic chat functionality
- [ ] Test UI customizations
- [ ] Test new upstream features

---

## Commit and Push

```bash
# Commit the merge
git add .
git commit -m "merge: upstream ${TARGET_RELEASE}

- Merged upstream Roo Code ${TARGET_RELEASE}
- Preserved Kilo Code UI customizations
- All tests passing"

# Create changeset
pnpm changeset
# Select "kilo-code", choose "minor" or "patch"

# Push
git push origin merge-upstream-${TARGET_RELEASE}
```

---

## Rollback

If merge causes issues:

```bash
# Option 1: Revert merge commit
git revert -m 1 <merge-commit-hash>
git push origin main

# Option 2: Reset to backup
git checkout backup-before-merge-YYYYMMDD
git checkout -b main-restored
git push origin main-restored --force
```

---

## Quick Reference

### Commands Cheatsheet

```bash
# Fetch and merge
git fetch upstream
git checkout -b merge-upstream-v1.2.3
git merge v1.2.3

# Resolve UI conflicts
git apply .kilocode/patches/ui-changes.patch
git add webview-ui/

# Verify
pnpm test
pnpm check-types
pnpm build

# Commit
git add .
git commit -m "merge: upstream v1.2.3"
pnpm changeset
git push origin merge-upstream-v1.2.3
```

### File Priority

| Path                         | Priority  | Strategy                     |
| ---------------------------- | --------- | ---------------------------- |
| `webview-ui/**`              | Kilo Code | Keep ours + integrate theirs |
| `**/*kilocode*`              | Kilo Code | Always keep ours             |
| `cli/**`                     | Kilo Code | Always keep ours             |
| `jetbrains/**`               | Kilo Code | Always keep ours             |
| Other with `kilocode_change` | Kilo Code | Preserve markers             |
| Other                        | Upstream  | Prefer theirs                |

---

**End of Guide**
