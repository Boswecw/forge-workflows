# Contributing to forge-workflows

Thank you for helping improve the BDS ecosystem CI/CD!

---

## ðŸ“‹ Before You Start

### What This Repo Is
forge-workflows provides reusable GitHub Actions workflows for Tauri applications (cortex_bds, VibeForge_BDS, etc.).

### What This Repo Is NOT
- It's not a place to add app-specific CI logic (that belongs in the calling repos)
- It's not for frontend-only or backend-only frameworks (it's Tauri-specific)

---

## ðŸ”„ Workflow Types

### 1. Bug Fixes
**Example:** "Clippy warnings appear incorrectly"

```bash
# Create branch
git checkout -b fix/clippy-warnings-false-positive
# Make changes
# Test in a calling repo
cd ../cortex_bds
git fetch origin
git checkout main

# Update ci.yml to use your branch (instead of @v1.0.0)
# uses: Boswecw/forge-workflows/.../tauri-ci-reusable.yml@fix/clippy-warnings-false-positive

git add .github/workflows/ci.yml
git commit -m "test: use fix branch for workflows"
git push origin main

# Verify CI passes
# Then back in forge-workflows, create PR
```

### 2. Feature Requests
**Example:** "Add support for code signing certificates"

**Before opening an issue:**
1. Check if it benefits 2+ repos (otherwise keep it app-specific)
2. Is it backwards compatible? (minor feature) or requires migration (major feature)
3. How long will it take to implement?

**Template:**
```markdown
## Feature Request: [Title]

### Problem
Why do we need this?

### Proposed Solution
How would this work?

### Impact
Which repos benefit? How much time saved?

### Example
How would a calling repo use it?
```

---

## ðŸ§ª Testing Your Changes

**Never merge changes without testing in a calling repo.**

### Step 1: Test Locally
```bash
cd forge-workflows
# Make changes to .github/workflows/tauri-ci-reusable.yml
vim .github/workflows/tauri-ci-reusable.yml
git add .github/workflows/tauri-ci-reusable.yml
git commit -m "feat: add new feature"
git push origin your-branch
```

### Step 2: Test in a Calling Repo
```bash
cd ../cortex_bds

# Create test branch
git checkout -b test/new-workflow-feature

# Update ci.yml to use your test branch
cat > .github/workflows/ci.yml << 'EOF'
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
jobs:
  tauri-ci:
    uses: Boswecw/forge-workflows/.github/workflows/tauri-ci-reusable.yml@your-branch
    with:
      tauri_workdir: src-tauri
      build_frontend: false
      run_tauri_build: false
EOF

git add .github/workflows/ci.yml
git commit -m "test: verify new workflow feature"
git push origin test/new-workflow-feature

# Open PR and wait for CI to run
# âœ… If CI passes, the feature is good
# âŒ If CI fails, debug and iterate
```

### Step 3: Merge to forge-workflows
Once calling repo's CI passes:
```bash
cd ../forge-workflows
# Open PR with your feature
# Request review
# Once approved, merge to main
```

### Step 4: Tag Release
```bash
cd forge-workflows
git checkout main
git pull origin main

# Decide version (major/minor/patch)
# For bug fixes: v1.0.1
# For new features: v1.1.0
# For breaking changes: v2.0.0

git tag v1.1.0
git push origin v1.1.0
```

### Step 5: Update Calling Repos
```bash
cd ../cortex_bds
# Update ci.yml and release.yml
sed -i 's/@v1.0.0/@v1.1.0/g' .github/workflows/*.yml
git add .github/workflows/
git commit -m "ci: update to shared workflows v1.1.0"
git push origin main
```

---

## âœ… Checklist Before Opening PR

- [ ] **Branch name follows pattern:** `fix/`, `feat/`, `docs/`, `chore/`
- [ ] **Tested in a calling repo** (cortex_bds or VibeForge_BDS)
- [ ] **Calling repo CI passed** (not just this repo)
- [ ] **Backwards compatible** (or clearly documented if breaking)
- [ ] **Updated README.md** if adding new inputs
- [ ] **No hardcoded paths** (use `inputs.tauri_workdir` etc.)
- [ ] **No app-specific logic** (workflows should be generic)
- [ ] **Clear commit message** (what changed and why)

---

## ðŸ“ Commit Message Format

```
[type]: [short description]

[longer explanation if needed]

Fixes #123 (if fixing an issue)
Tested-in: cortex_bds, VibeForge_BDS
```

**Types:**
- `fix:` Bug fix
- `feat:` New feature
- `docs:` Documentation
- `chore:` Maintenance

**Examples:**
```
fix: correct Clippy false positive for unused variable

The workflow was reporting warnings that don't exist. 
Updated clippy invocation to use correct filter.

Tested-in: cortex_bds âœ…
```

```
feat: add support for custom Rust targets

Allows calling repos to specify additional Rust 
targets for cross-compilation.

Input: rust_targets (string, comma-separated)
Example: "x86_64-unknown-linux-musl,aarch64-unknown-linux-gnu"

Tested-in: cortex_bds, VibeForge_BDS âœ…
```

---

## ðŸ”’ Breaking Changes

If your change is breaking (e.g., renames an input or changes behavior):

1. **Document clearly** in PR description
2. **Include migration guide** for calling repos
3. **Tag major version** (v2.0.0)
4. **Notify all teams** that use these workflows

**Example Migration Guide:**
```markdown
## Breaking Change in v2.0.0

### What Changed
Renamed input `frontend_build` â†’ `build_frontend` (boolean semantics match naming)

### Migration Steps
In your `.github/workflows/ci.yml`:

```yaml
# Before
uses: .../tauri-ci-reusable.yml@v1.0.0
with:
  frontend_build: true

# After
uses: .../tauri-ci-reusable.yml@v2.0.0
with:
  build_frontend: true
```

No other changes needed. All other inputs remain the same.
```

---

## ðŸ“ž Questions?

1. **Is this a good idea for the shared repo?** Check if it benefits 2+ apps
2. **How do I test this?** Follow the "Testing Your Changes" section above
3. **What if something breaks?** Use the calling repo's `git checkout main` to rollback, then debug in your feature branch

---

## ðŸ“Š Good Examples of Contributions

âœ… **Bug fix:** "Clippy false positive on unused variable"  
âœ… **Feature:** "Add support for code signing secrets"  
âœ… **Optimization:** "Add pnpm caching to speed up builds 30%"  
âœ… **Documentation:** "Add troubleshooting section to README"  

âŒ **Bad:** "Add cortex_bds-specific linting rules" (app-specific)  
âŒ **Bad:** "Support Python projects" (Tauri-only scope)  
âŒ **Bad:** "Add Slack notification" (without testing)  

---

## ðŸš€ Code Review

When your PR is reviewed:

1. **Request will include:** Specific questions or requested changes
2. **Timeline:** Expect feedback within 1-2 days
3. **Multiple reviewers:** Major features may require 2+ approvals
4. **CI must pass:** Your PR's CI must be green before merge

---

## ðŸ“š Related Files

- **README.md** â€” Usage guide for calling repos
- **tauri-ci-reusable.yml** â€” Main CI workflow
- **tauri-release-reusable.yml** â€” Release workflow
- **CONTRIBUTING.md** â€” This file

---

**Thank you for improving the BDS ecosystem! ðŸš€**
