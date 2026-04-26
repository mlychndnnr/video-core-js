# Automatic NPM Publishing - Setup Complete ✅

## What Changed

The NPM publishing workflow has been updated to **automatically trigger** when a new release is created by semantic-release.

---

## New Automated Flow

```
Developer merges PR to stable
         ↓
Release workflow creates GitHub release (with tag)
         ↓
NPM publish workflow AUTOMATICALLY triggers
         ↓
Package published to NPM
         ↓
✅ Done! Users can install new version
```

**Timeline:** ~5-10 minutes from PR merge to NPM availability

---

## What Was Updated

### 1. **npm-publish.yml Workflow**

**File:** `.github/workflows/npm-publish.yml`

#### Added Automatic Trigger:
```yaml
on:
  release:
    types:
      - published  # ← Triggers when release.yml creates a release
```

#### Still Supports Manual Trigger:
```yaml
  workflow_dispatch:  # Can still trigger manually if needed
    inputs:
      s3-path: ...
      file-to-upload: ...
```

#### Added Summary Steps:
- Log which trigger was used (automatic vs manual)
- Show NPM URL after publish
- Display installation command

---

## Complete Workflow Sequence

### Step-by-Step:

1. **Developer Creates PR**
   ```bash
   git checkout -b fix/bug-fix
   git commit -m "fix: resolve calculation error"
   git push origin fix/bug-fix
   # Create PR to stable
   ```

2. **PR Review & Tests**
   - ✅ test.yml runs on PR
   - ✅ Code review completed
   - ✅ Tests pass

3. **PR Merged to Stable**
   ```
   PR merged → Triggers release.yml
   ```

4. **Release Workflow (release.yml)**
   - ✅ Analyzes commits
   - ✅ Determines version bump (4.1.5 → 4.1.6)
   - ✅ Updates package.json
   - ✅ Updates changelog.md
   - ✅ Creates git tag v4.1.6
   - ✅ Creates GitHub release
   - ✅ Commits changes back

5. **NPM Publish Workflow (npm-publish.yml) - AUTOMATIC**
   - ✅ Detects release published event
   - ✅ Checks out code
   - ✅ Installs dependencies
   - ✅ Builds project
   - ✅ Runs security audit
   - ✅ Publishes to NPM with correct tag (latest/beta)
   - ✅ Uploads to S3 (if configured)

6. **Users Can Install**
   ```bash
   npm install @newrelic/video-core@4.1.6
   # or
   npm install @newrelic/video-core@latest
   ```

---

## Triggers Comparison

| Trigger Type | When | Why |
|--------------|------|-----|
| **Automatic (release)** | When release.yml creates a release | Default - fastest delivery |
| **Manual (workflow_dispatch)** | When manually triggered | Override or special cases |
| **Workflow call** | Called by another workflow | Reusable workflow pattern |

---

## Safety Measures

Even with automatic publishing, multiple safety checks are in place:

### Before Release Creation:
1. ✅ PR must be reviewed and approved
2. ✅ Tests must pass (test.yml)
3. ✅ Conventional commit format validated
4. ✅ Build must succeed

### Before NPM Publish:
5. ✅ Dependencies installed and verified
6. ✅ Project builds successfully
7. ✅ Security audit passes (critical vulnerabilities block publish)
8. ✅ Correct NPM tag determined (latest vs beta)

### After Publish:
9. ✅ Summary shows NPM URL and install command
10. ✅ GitHub Actions logs provide full audit trail

---

## Benefits of Automatic Publishing

### 1. **Speed** ⚡
```
Before: PR merge → wait → manual trigger → publish (hours/days)
Now:    PR merge → automatic publish (5-10 minutes)
```

### 2. **Consistency** 🎯
- Every release automatically goes to NPM
- No forgotten publishes
- Predictable process

### 3. **Less Overhead** 🚀
- No need to remember to publish
- No manual steps after PR merge
- Developers focus on code, not releases

### 4. **Faster Bug Fixes** 🐛
```
Bug found → Fix merged → Automatically on NPM in minutes
vs
Bug found → Fix merged → Wait for manual publish → Eventually on NPM
```

---

## When Manual Trigger Is Still Useful

You can still manually trigger npm-publish.yml for:

### 1. **Re-publishing Failed Release**
If NPM publish fails (network, credentials, etc.):
```
Actions → Publish Module → Run workflow
```

### 2. **Publishing with S3 Upload**
Manually trigger with S3 parameters:
```
Actions → Publish Module → Run workflow
  s3-path: browser-agent/video
  file-to-upload: dist/umd/video-core.min.js
```

### 3. **Testing Before Automation**
Test publish process before enabling automatic trigger

### 4. **Emergency Override**
Publish specific version without creating new release

---

## Monitoring Releases

### Check Release Status:

1. **GitHub Releases**
   ```
   https://github.com/newrelic/video-core-js/releases
   ```

2. **GitHub Actions**
   ```
   https://github.com/newrelic/video-core-js/actions
   ```
   - Check "Release" workflow for version creation
   - Check "Publish Module" workflow for NPM publish

3. **NPM Registry**
   ```
   https://www.npmjs.com/package/@newrelic/video-core
   ```

---

## Rollback / Emergency

### If Bad Version Published:

1. **Don't panic** - NPM versions are immutable
2. **Quick fix approach:**
   ```bash
   # Fix the issue
   git commit -m "fix: urgent fix for v4.1.6 issue"
   git push origin stable
   # This will automatically create v4.1.7
   ```

3. **Deprecate bad version (if needed):**
   ```bash
   npm deprecate @newrelic/video-core@4.1.6 "Issue found, use 4.1.7+"
   ```

4. **Communication:**
   - Update GitHub release notes
   - Notify users of issue
   - Point to fixed version

---

## Testing the Automatic Flow

### Test in Your Fork:

The testing fork at `/Users/cmalay/Desktop/Workflow/video-core-js` can be updated with the same automatic trigger:

1. Copy the updated npm-publish.yml
2. Configure NPM Trusted Publishing for your fork
3. Test the end-to-end flow
4. Verify automatic trigger works

---

## Configuration Files

### Key Files:
```
.github/workflows/
├── release.yml          # Creates releases automatically
├── npm-publish.yml      # Publishes to NPM automatically (UPDATED)
└── test.yml             # Runs tests on PRs

.releaserc.json          # Semantic-release configuration
package.json             # Package version (auto-updated)
changelog.md             # Changelog (auto-updated)
WORKFLOW.md              # Documentation (UPDATED)
```

---

## Commit Message Impact

Remember: Your commit messages now directly trigger releases AND NPM publishes:

| Commit | Version | Release? | NPM Publish? |
|--------|---------|----------|--------------|
| `fix: bug` | 4.1.5 → 4.1.6 | ✅ Yes | ✅ Yes (automatic) |
| `feat: feature` | 4.1.6 → 4.2.0 | ✅ Yes | ✅ Yes (automatic) |
| `feat!: breaking` | 4.2.0 → 5.0.0 | ✅ Yes | ✅ Yes (automatic) |
| `docs: update` | No change | ❌ No | ❌ No |
| `chore: deps` | No change | ❌ No | ❌ No |

**Be extra careful with commit messages!**

---

## Best Practices

### 1. **Thorough PR Reviews**
Since publishing is automatic, PR review is critical:
- ✅ Code quality check
- ✅ Test coverage
- ✅ Commit message format
- ✅ Breaking changes documented

### 2. **Good Commit Messages**
Follow conventional commits strictly:
```bash
# Good
fix: resolve bitrate calculation error
feat: add QoE metrics tracking
feat!: remove deprecated API

# Bad
Fixed bug
Added feature
Updates
```

### 3. **Test Coverage**
Maintain high test coverage to catch issues before publish:
```bash
npm test  # Should pass before merging
```

### 4. **Monitor Releases**
Watch GitHub Actions after PR merge:
- Check release.yml succeeds
- Check npm-publish.yml succeeds
- Verify package on NPM

---

## Reverting to Manual (If Needed)

If automatic publishing doesn't fit your workflow:

### 1. Edit npm-publish.yml:
```yaml
# Remove automatic trigger
on:
  # Delete this section:
  # release:
  #   types:
  #     - published

  # Keep only manual trigger:
  workflow_dispatch:
    inputs: ...
```

### 2. Update WORKFLOW.md

### 3. Communicate change to team

---

## Summary

✅ **Automatic NPM publishing is now ACTIVE**

📝 **What to remember:**
- PR merge → Release created → NPM published (all automatic)
- Manual trigger still available if needed
- Safety checks remain in place
- Commit messages are critical

🚀 **Result:**
Faster delivery, less manual work, consistent releases!

---

## Questions?

- **Configuration**: See `.github/workflows/npm-publish.yml`
- **Full workflow**: See `WORKFLOW.md`
- **Testing**: Use fork at `/Users/cmalay/Desktop/Workflow/video-core-js`
- **Issues**: Check GitHub Actions logs

**Status:** ✅ Setup Complete and Ready!
