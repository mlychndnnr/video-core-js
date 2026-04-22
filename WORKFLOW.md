# Complete Release Workflow Guide for Beginners

This document explains **step-by-step** how the automated release process works in this repository, from making code changes to publishing on NPM.

## Table of Contents
1. [Overview](#overview)
2. [The Complete Flow](#the-complete-flow)
3. [Step-by-Step Detailed Explanation](#step-by-step-detailed-explanation)
4. [Understanding Semantic Versioning](#understanding-semantic-versioning)
5. [Commit Message Format](#commit-message-format)
6. [Practical Examples](#practical-examples)
7. [Workflows Explained](#workflows-explained)

---

## Overview

We use **semantic-release** to automate our release process. This means:
- ✅ No manual version updates needed
- ✅ Automatic changelog generation
- ✅ Consistent release process
- ✅ Version numbers follow semantic versioning rules

---

## The Complete Flow

Here's the **big picture** of what happens from code change to NPM publish:

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. DEVELOPER MAKES CHANGES                                      │
│    - Write code in feature/fix branch                          │
│    - Write tests                                                │
│    - Commit with conventional commit message                   │
│    - Create Pull Request to stable branch                      │
│    - Get PR approved and merge to stable                       │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. RELEASE WORKFLOW TRIGGERS (release.yml)                      │
│    - Automatically runs when PR is merged to stable            │
│    - Analyzes commit messages                                  │
│    - Determines next version number                            │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. VERSION & FILES GET UPDATED                                  │
│    Step 3a: Update package.json (e.g., 4.1.5 → 4.2.0)         │
│    Step 3b: Update package-lock.json                           │
│    Step 3c: Generate and update changelog.md                   │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. CREATE GIT TAG & GITHUB RELEASE                             │
│    Step 4a: Create Git tag (e.g., v4.2.0)                     │
│    Step 4b: Push tag to GitHub                                 │
│    Step 4c: Create GitHub Release with notes                   │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 5. COMMIT CHANGES BACK TO REPOSITORY                           │
│    - Commit updated files (package.json, changelog.md)        │
│    - Push commit with [skip ci] to avoid infinite loop        │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 6. MANUAL: TRIGGER NPM PUBLISH WORKFLOW                        │
│    - Developer reviews the release                             │
│    - Manually triggers npm-publish.yml workflow                │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 7. NPM PUBLISH WORKFLOW RUNS (npm-publish.yml)                 │
│    Step 7a: Checkout code with new version                    │
│    Step 7b: Install dependencies                               │
│    Step 7c: Build project (creates dist/ folder)              │
│    Step 7d: Run security audit                                 │
│    Step 7e: Publish to NPM registry                            │
│    Step 7f: Upload to S3 (if configured)                       │
└─────────────────────────────────────────────────────────────────┘
                            ↓
                    ✅ RELEASE COMPLETE!
```

---

## Step-by-Step Detailed Explanation

### **STEP 1: Developer Makes Changes**

**What happens:**
You write code and commit it using a special commit message format.

**Example:**
```bash
# You fixed a bug in the video tracker on a feature branch
git checkout -b fix/bitrate-calculation
git add src/videotracker.js
git commit -m "fix: correct bitrate calculation for live streams"
git push origin fix/bitrate-calculation

# Then create a Pull Request to stable branch and get it merged
```

**Why this matters:**
The commit message `fix:` tells semantic-release that this is a bug fix, which will trigger a **patch** version bump (e.g., 4.1.5 → 4.1.6) when the PR is merged to `stable`.

---

### **STEP 2: Release Workflow Automatically Triggers**

**What happens:**
When a Pull Request is **merged** to the `stable` branch, GitHub Actions automatically runs the `release.yml` workflow.

**File:** `.github/workflows/release.yml`

**What it does:**
1. Checks out your code
2. Installs dependencies (`npm ci`)
3. Runs tests (`npm test`)
4. Runs build (`npm run build`)
5. Calls semantic-release

**In the logs, you'll see:**
```
✓ Run tests
✓ Run build
✓ Verify .releaserc.json exists
✓ Run semantic-release
```

---

### **STEP 3: Semantic-Release Analyzes & Updates Files**

**What happens:**
Semantic-release reads your commit messages and updates version files.

#### **Step 3a: Analyze Commits**
- Reads all commits since last release
- Looks for keywords: `feat:`, `fix:`, `BREAKING CHANGE:`
- Determines version bump type

**Example:**
```
Commits found:
- fix: correct bitrate calculation
- fix: handle null values in tracker
- docs: update README

Decision: PATCH version bump (4.1.5 → 4.1.6)
```

#### **Step 3b: Update package.json**
**Before:**
```json
{
  "name": "@newrelic/video-core",
  "version": "4.1.5",
  ...
}
```

**After:**
```json
{
  "name": "@newrelic/video-core",
  "version": "4.1.6",
  ...
}
```

#### **Step 3c: Update package-lock.json**
Updates the version field in package-lock.json to match package.json.

**Before:**
```json
{
  "name": "@newrelic/video-core",
  "version": "4.1.5",
  "lockfileVersion": 3,
  ...
}
```

**After:**
```json
{
  "name": "@newrelic/video-core",
  "version": "4.1.6",
  "lockfileVersion": 3,
  ...
}
```

#### **Step 3d: Update changelog.md**
Generates release notes and adds them to the changelog.

**What gets added:**
```markdown
# Changelog

## [4.1.6] - 2026/04/23

### Bug Fixes

- correct bitrate calculation for live streams
- handle null values in tracker

## [4.1.5] - 2026/04/08
...
```

---

### **STEP 4: Create Git Tag & GitHub Release**

#### **Step 4a: Create Git Tag**
**What happens:**
A git tag is created with the version number.

```bash
git tag v4.1.6
```

**Why tags matter:**
- Tags mark specific points in your repository's history
- They make it easy to reference specific releases
- NPM and other tools use tags to identify versions

#### **Step 4b: Push Tag to GitHub**
```bash
git push origin v4.1.6
```

#### **Step 4c: Create GitHub Release**
**What happens:**
A GitHub Release is created with:
- Release title: "v4.1.6"
- Release notes from changelog
- Link to compare changes with previous version

**You can see it at:**
`https://github.com/newrelic/video-core-js/releases/tag/v4.1.6`

---

### **STEP 5: Commit Changes Back to Repository**

**What happens:**
Semantic-release commits the updated files back to your repository.

**Files committed:**
- `package.json` (updated version)
- `package-lock.json` (updated version)
- `changelog.md` (new release notes)

**Commit message:**
```
chore(release): 4.1.6 [skip ci]

## [4.1.6] - 2026/04/23

### Bug Fixes
- correct bitrate calculation for live streams
```

**Important:** The `[skip ci]` prevents an infinite loop by telling GitHub Actions not to run workflows on this commit.

---

### **STEP 6: Manual Trigger of NPM Publish**

**What happens:**
⚠️ **This step is MANUAL** - you need to trigger it yourself.

**Why manual?**
- Gives you a chance to review the release
- Ensures you're ready to publish to NPM
- Prevents accidental publishes

**How to trigger:**
1. Go to GitHub Actions tab
2. Click on "Publish Module" workflow
3. Click "Run workflow"
4. Confirm and run

**Or via Pull Request:**
- Create a PR to `stable` or `stable-beta` branch
- When merged, it automatically triggers the publish workflow

---

## Manual vs Automatic NPM Publishing

This repository uses **manual NPM publishing** by default. Here's a comparison to help you understand the trade-offs:

### 📋 Manual NPM Publishing (Current Setup)

#### ✅ Pros:

1. **Safety & Control**
   - Review the release before it goes live to users
   - Verify version number, changelog, and release notes are correct
   - Check that tests passed and build is successful
   - Catch any last-minute issues before publishing

2. **Two-Step Verification**
   - Step 1: Create release (automatic) - updates version, creates tag
   - Step 2: Publish to NPM (manual) - makes package available to users
   - If something is wrong, you can fix it before users see it

3. **Staging Period**
   - Time to test the release internally before going public
   - Time to prepare announcements or documentation
   - Time to coordinate with your team or stakeholders

4. **Prevents Irreversible Mistakes**
   - Once published to NPM, you **cannot unpublish** (NPM policy)
   - You can only deprecate versions (which looks bad)
   - Manual trigger prevents accidental publishes of broken code

5. **Compliance & Approval**
   - Some organizations require manual approval before public releases
   - Audit trail of who triggered the publish
   - Ability to hold releases for business reasons

#### ❌ Cons:

1. **Extra Step Required**
   - Must manually trigger the publish workflow after release
   - Requires someone to remember to publish

2. **Potential Delays**
   - Release is created but not immediately available on NPM
   - Users have to wait for manual publish step

3. **Human Oversight Needed**
   - Someone needs to monitor releases and trigger publishes
   - Can be forgotten during holidays or off-hours

4. **Not Fully Automated**
   - Defeats some benefits of CI/CD automation
   - Additional manual intervention in the pipeline

#### 📖 Example Scenario (Manual):
```
✅ Good outcome:
PR merged → Release v4.2.0 created → You review → Found typo in changelog
→ Fix changelog → Publish to NPM → Clean release!

❌ Without manual step:
PR merged → Release v4.2.0 created → Auto-published → Found typo
→ Too late! Already on NPM with wrong changelog
```

---

### 🤖 Automatic NPM Publishing (Alternative)

#### ✅ Pros:

1. **Fully Automated**
   - No manual intervention required
   - True continuous delivery (CD)
   - Faster time from code merge to NPM availability

2. **Consistency**
   - Every release automatically goes to NPM
   - No human error or forgotten publishes
   - Predictable release process

3. **Speed**
   - Users get updates immediately after PR merge
   - No waiting for manual publish step
   - Faster bug fix delivery

4. **Less Overhead**
   - No need to monitor and manually publish
   - Works 24/7, even on weekends/holidays
   - Frees up developer time

#### ❌ Cons:

1. **Less Control**
   - Cannot review release before it goes live
   - No staging period to catch issues
   - Once merged, it's immediately public

2. **Higher Risk**
   - Broken code goes straight to NPM users
   - No safety net before publishing
   - Mistakes are immediately public

3. **Cannot Hold Releases**
   - Cannot delay publishing for business reasons
   - Cannot coordinate release timing
   - No ability to batch multiple fixes

4. **Irreversible Publishes**
   - NPM doesn't allow unpublishing packages
   - Must publish a new patch version to fix issues
   - Version numbers get wasted on broken releases

5. **Trust in CI/CD Required**
   - Must have 100% confidence in tests
   - Must have comprehensive test coverage
   - One bad merge = public broken release

#### 📖 Example Scenario (Automatic):
```
❌ Bad outcome:
PR merged → Release v4.2.0 created → Auto-published → Users report bug
→ Must release v4.2.1 immediately → Emergency fix → Public embarrassment

✅ With good tests:
PR merged → Release v4.2.0 created → Auto-published
→ Works perfectly → Users happy!
```

---

### 🔄 Hybrid Approach Options

#### Option 1: Conditional Automation
- **Patch/Minor versions**: Auto-publish (low risk)
- **Major versions**: Manual review (breaking changes)

```yaml
if: startsWith(github.ref, 'refs/tags/v') && !contains(github.ref, '.0.0')
```

#### Option 2: Branch-Based
- **stable branch**: Manual publish (production)
- **stable-beta branch**: Auto-publish with `beta` tag (testing)

#### Option 3: Approval Required
- Use GitHub Environments with required reviewers
- Automatic but requires approval from designated team members
- Best of both worlds: automated + safety

---

### 📊 Recommendation

**For this repository, we use Manual Publishing because:**

1. ✅ **Safety First**: NPM packages affect many users
2. ✅ **Quality Control**: Review before public release
3. ✅ **Compliance**: Enterprise requirements for approval
4. ✅ **Flexibility**: Can hold or coordinate releases

**When to Consider Automatic:**
- High test coverage (>90%)
- Well-established CI/CD practices
- Small team needing speed over control
- Internal packages (not public NPM)
- Beta/canary releases

---

### 🔧 Switching to Automatic (If Needed)

If you decide to switch to automatic NPM publishing, you can:

1. **Update npm-publish.yml** to trigger on release creation
2. **Add environment protection** for approval gates
3. **Set up conditional logic** for version-based automation

See the repository maintainer or DevOps team to discuss changing this configuration.

---

---

### **STEP 7: NPM Publish Workflow Runs**

**File:** `.github/workflows/npm-publish.yml`

This workflow publishes your package to NPM and optionally uploads files to S3.

#### **Step 7a: Checkout Code**
```bash
git checkout master
```
Gets the latest code with the updated version from Step 5.

#### **Step 7b: Setup Node.js**
```bash
node-version: '24.x'
```
Installs Node.js 24.x which supports NPM Trusted Publishing.

#### **Step 7c: Install Dependencies**
```bash
npm ci
```
- `npm ci` (clean install) is like `npm install` but faster and more reliable
- It uses the exact versions from `package-lock.json`

#### **Step 7d: Build Project**
```bash
npm run build
```

**What this does:**
- Runs webpack to bundle your code
- Creates the `dist/` folder with:
  - `dist/cjs/` - CommonJS format (for Node.js)
  - `dist/esm/` - ES Module format (for modern bundlers)
  - `dist/umd/` - Universal Module Definition (for browsers)

**Example output:**
```
dist/
├── cjs/
│   └── index.js
├── esm/
│   └── index.js
└── umd/
    └── video-core.min.js
```

#### **Step 7e: Get Package Version**
```bash
PACKAGE_VERSION=$(jq -r '.version' package.json)
```

Reads version from package.json (e.g., "4.1.6")

**Determines NPM tag:**
- If version contains "beta" → uses `beta` tag
- Otherwise → uses `latest` tag

**Example:**
- `4.1.6` → published as `@newrelic/video-core@latest`
- `4.2.0-beta.1` → published as `@newrelic/video-core@beta`

#### **Step 7f: Run Security Audit**
```bash
npm audit --audit-level=critical
```

Checks for critical security vulnerabilities before publishing.

#### **Step 7g: Publish to NPM**
```bash
npm publish --tag latest
```

**What happens:**
1. NPM packages your files based on `package.json` "files" field
2. Uploads to NPM registry at registry.npmjs.org
3. Uses NPM Trusted Publishing (OIDC) for secure authentication

**What gets published:**
```
@newrelic/video-core@4.1.6
├── dist/
├── src/
├── CHANGELOG.md
├── LICENSE
├── README.md
└── package.json
```

**Users can now install:**
```bash
npm install @newrelic/video-core@4.1.6
# or
npm install @newrelic/video-core@latest
```

#### **Step 7h: Upload to S3 (Optional)**

**Only runs if:** `s3-path` input is provided

**What happens:**
```bash
# Uploads two versions of the file:
# 1. Versioned file: video-core.4.1.6.min.js
# 2. Latest file: video-core.latest.min.js

aws s3 cp video-core.4.1.6.min.js s3://nr-downloads-main/path/
aws s3 cp video-core.latest.min.js s3://nr-downloads-main/path/
```

**Why two files?**
- **Versioned**: Users can reference specific versions
- **Latest**: Always points to the newest version

---

## Understanding Semantic Versioning

Version numbers follow the format: **MAJOR.MINOR.PATCH**

Example: `4.1.6`
- **4** = Major version
- **1** = Minor version
- **6** = Patch version

### When to bump each number:

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| **Breaking Change** | MAJOR | 4.1.6 → **5.0.0** |
| **New Feature** | MINOR | 4.1.6 → **4.2.0** |
| **Bug Fix** | PATCH | 4.1.6 → **4.1.7** |

### Examples:

**Patch (Bug Fix):**
```
4.1.5 → 4.1.6
```
- Small bug fixes
- No new features
- No breaking changes
- Safe to upgrade

**Minor (New Feature):**
```
4.1.6 → 4.2.0
```
- New features added
- Backwards compatible
- No breaking changes
- Safe to upgrade

**Major (Breaking Change):**
```
4.2.0 → 5.0.0
```
- Breaking changes
- API changes
- Removed features
- Requires code changes to upgrade

---

## Commit Message Format

### Basic Format:
```
<type>: <description>

[optional body]

[optional footer]
```

### Types and Their Impact:

| Type | Description | Version Bump | Example |
|------|-------------|--------------|---------|
| `feat:` | New feature | **MINOR** | `feat: add QoE tracking` |
| `fix:` | Bug fix | **PATCH** | `fix: resolve null pointer` |
| `perf:` | Performance improvement | **PATCH** | `perf: optimize bitrate calc` |
| `docs:` | Documentation | **NONE** | `docs: update README` |
| `chore:` | Maintenance | **NONE** | `chore: update deps` |
| `refactor:` | Code refactoring | **NONE** | `refactor: simplify tracker` |
| `test:` | Tests | **NONE** | `test: add unit tests` |
| `style:` | Formatting | **NONE** | `style: fix indentation` |
| `ci:` | CI changes | **NONE** | `ci: update workflow` |

### Breaking Changes:

Add `!` after type or add `BREAKING CHANGE:` in footer:

```
feat!: remove deprecated methods

BREAKING CHANGE: Removed getRenditionBitrate() method.
Use getSegmentDownloadBitrate() instead.
```

This triggers a **MAJOR** version bump.

---

## Practical Examples

### Example 1: Bug Fix Release

**Scenario:** You fixed a bug where bitrate was calculated incorrectly.

**Steps:**
```bash
# 1. Create a fix branch
git checkout -b fix/bitrate-calculation

# 2. Fix the bug
vim src/videotracker.js

# 3. Commit with 'fix:' prefix
git add src/videotracker.js
git commit -m "fix: correct bitrate calculation for live streams"

# 4. Push the branch
git push origin fix/bitrate-calculation

# 5. Create PR to stable and get it merged
```

**What happens automatically:**
1. ✅ Release workflow triggers
2. ✅ Tests run and pass
3. ✅ Version bumps: 4.1.5 → **4.1.6** (patch)
4. ✅ package.json updated
5. ✅ changelog.md updated
6. ✅ Git tag `v4.1.6` created
7. ✅ GitHub release created

**Manual step:**
8. 👤 You review the release
9. 👤 You trigger "Publish Module" workflow
10. ✅ Package published to NPM

---

### Example 2: New Feature Release

**Scenario:** You added a new QoE metrics feature.

**Steps:**
```bash
# 1. Create a feature branch
git checkout -b feat/qoe-metrics

# 2. Add the feature
vim src/qoe-tracker.js

# 3. Commit with 'feat:' prefix
git add src/qoe-tracker.js
git commit -m "feat: add real-time QoE health metrics tracking"

# 4. Push the branch
git push origin feat/qoe-metrics

# 5. Create PR to stable and get it merged
```

**What happens automatically:**
1. ✅ Release workflow triggers
2. ✅ Version bumps: 4.1.6 → **4.2.0** (minor)
3. ✅ Files updated and committed
4. ✅ Tag and release created

---

### Example 3: Breaking Change Release

**Scenario:** You removed a deprecated API method.

**Steps:**
```bash
# 1. Create a feature branch
git checkout -b feat/remove-deprecated-api

# 2. Remove old method
vim src/videotracker.js

# 3. Commit with 'feat!' and BREAKING CHANGE
git add src/videotracker.js
git commit -m "feat!: remove deprecated getRenditionBitrate method

BREAKING CHANGE: Removed getRenditionBitrate() method.
Use getSegmentDownloadBitrate() instead."

# 4. Push the branch
git push origin feat/remove-deprecated-api

# 5. Create PR to stable and get it merged
```

**What happens automatically:**
1. ✅ Release workflow triggers
2. ✅ Version bumps: 4.2.0 → **5.0.0** (major)
3. ✅ Changelog includes breaking change warning
4. ✅ Tag and release created

---

### Example 4: Documentation Update (No Release)

**Scenario:** You updated the README.

**Steps:**
```bash
# 1. Create a docs branch
git checkout -b docs/qoe-examples

# 2. Update docs
vim README.md

# 3. Commit with 'docs:' prefix
git add README.md
git commit -m "docs: add examples for QoE tracking"

# 4. Push and merge to stable
git push origin docs/qoe-examples
# Create PR and merge
```

**What happens:**
1. ❌ No release triggered (docs don't affect version)
2. ✅ Code merged to stable
3. ❌ No version bump
4. ❌ No changelog update

---

## Workflows Explained

### 1. release.yml (Automated Release)

**Triggers:**
- Automatically on push to `master`
- Manually via "Run workflow" button

**What it does:**
1. Runs tests
2. Analyzes commits
3. Updates version files
4. Creates tag and release

**When to use:**
- Runs automatically - you don't need to do anything!
- Use manual trigger for dry-run testing

---

### 2. npm-publish.yml (NPM Publishing)

**Triggers:**
- Manually via "Run workflow" button
- Automatically when PR merged to `stable` or `stable-beta`

**What it does:**
1. Builds the project
2. Publishes to NPM
3. Uploads to S3 (optional)

**When to use:**
- After reviewing a release
- When you're ready to make the package available on NPM

---

### 3. test.yml (Testing)

**Triggers:**
- On pull requests
- On push to `stable` or `develop` branches

**What it does:**
1. Runs `npm test`
2. Shows test coverage

**When to use:**
- Automatically runs on PRs
- Ensures code quality before merging

---

## Quick Reference

### Release Checklist

- [ ] Make your code changes
- [ ] Write tests for your changes
- [ ] Commit with conventional commit message
- [ ] Push to master branch
- [ ] Wait for release workflow to complete
- [ ] Review the GitHub release
- [ ] Trigger "Publish Module" workflow
- [ ] Verify package on NPM
- [ ] Announce release to team

### Common Commands

```bash
# Check current version
cat package.json | grep version

# View recent commits
git log --oneline -10

# View all tags
git tag -l

# Preview what semantic-release will do
npx semantic-release --dry-run

# Install your package
npm install @newrelic/video-core@latest
```

---

## Troubleshooting

### Problem: Release workflow didn't trigger
**Solution:** Check that:
- Your PR was merged to `stable` branch (not just closed)
- Your commit message follows conventional format
- The commit type triggers a release (`feat:`, `fix:`, `perf:`)

### Problem: Wrong version number
**Solution:** Review your commit message:
- `fix:` = patch (4.1.5 → 4.1.6)
- `feat:` = minor (4.1.5 → 4.2.0)
- `feat!:` or `BREAKING CHANGE:` = major (4.1.5 → 5.0.0)

### Problem: NPM publish failed
**Solution:** Check:
- You have NPM_TOKEN configured in GitHub secrets
- Package name is available on NPM
- Version doesn't already exist on NPM

---

## Summary

1. **Write code** → Commit with conventional message → Create PR to stable → Get merged
2. **Release workflow** → Analyzes commits → Updates version files
3. **Creates tag** → Creates GitHub release → Commits changes back
4. **Manual trigger** → Publish workflow → Publishes to NPM
5. **Done!** → Package available for users

This automated process ensures:
- ✅ Consistent versioning
- ✅ Accurate changelogs
- ✅ Reliable releases
- ✅ Less manual work
- ✅ Fewer errors

---

**Questions?** Check the configuration files:
- `.releaserc.json` - Semantic release config
- `.github/workflows/release.yml` - Release workflow
- `.github/workflows/npm-publish.yml` - Publish workflow
