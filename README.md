# Mobile App Release: Standard Operating Procedure (SOP)

**Document Version:** 1.2
**Last Updated:** May 22, 2025

## 1. Overview

### 1.1. Purpose
This SOP provides step-by-step instructions for releasing new versions and hotfixes for the mobile application. Follow these procedures precisely.

### 1.2. Versioning Convention
* **Format:** `MAJOR.MINOR.PATCH+BUILD_NUMBER` (e.g., `1.0.7+73`)
* **`MAJOR`**: Breaking changes / significant new features.
* **`MINOR`**: New, backward-compatible features.
* **`PATCH`**: Backward-compatible bug fixes.
* **`+BUILD_NUMBER`**: Unique, incrementing integer for each store/test build.
    * Android: `versionCode`
    * iOS: `CFBundleVersion` (Build)

### 1.3. Key Branches
* **`main`**: Production code.
* **`develop`**: Current development integration.
* **`release/<version>`**: Release preparation (e.g., `release/1.2.0`).
* **`hotfix/<fix-or-version>`**: Urgent production fixes.

### 1.4. Core Requirements
* **Pull Requests (PRs) & Code Reviews**: Mandatory for merges to `develop`, `main`, `release/*`, `hotfix/*`.
* **CI/CD**: All automated tests must pass before merging.
* **Changelog**: Update `CHANGELOG.md` for every release.

---
---

## 2. Standard Release Procedure

**Use this procedure for:** Planned Major, Minor, or Patch releases.
**Example Scenario:** Releasing version `1.2.0`. Current production is `1.1.5+60`. Initial build for this release branch will be `1.2.0+61`.

---

### Stage 1: Preparation & Branching

1.  **Verify `develop` Branch Readiness:**
    * Confirm all planned features for this release are merged into `develop`.
    * Ensure `develop` is stable and all CI checks are passing.
2.  **Announce Feature Freeze (Release Lead):**
    * Communicate that `develop` is now feature-frozen for version `1.2.0`.
    * Only critical bug fixes *for this specific release* may be merged to `develop` before branching.
3.  **Create `release` Branch from `develop`:**
    ```bash
    git checkout develop
    git pull origin develop
    git checkout -b release/1.2.0
    git push origin release/1.2.0
    ```
    * *Note: `develop` can now accept features for the *next* release cycle.*
4.  **Bump Version on `release/1.2.0`:**
    * Update app version to `1.2.0` and set initial `+BUILD_NUMBER` (e.g., `+61`).
    * Modify files: `pubspec.yaml`, Android `build.gradle`, iOS `Info.plist`.
    * Commit and push:
        ```bash
        git add . # Stage all changes
        git commit -m "chore: Bump version to 1.2.0+61 for release"
        git push origin release/1.2.0
        ```

---

### Stage 2: Stabilization & QA (on `release/1.2.0`)

1.  **Build & Distribute for QA:**
    * Generate a build from `release/1.2.0` (e.g., `1.2.0+61`).
    * Distribute to QA via TestFlight, Google Play Internal Testing, Firebase App Distribution, etc.
2.  **QA Testing & Bug Fixing:**
    * QA Team: Test the build thoroughly. Report bugs.
    * Developers: Fix reported bugs directly on the `release/1.2.0` branch.
        * Commit each fix: `fix: Description of fix (release/1.2.0)`
        * For each new build provided to QA from this branch (after fixes), **increment the `+BUILD_NUMBER`** (e.g., `1.2.0+62`, then `1.2.0+63`). Update version files, commit, and push.
3.  **Update `CHANGELOG.md`:**
    * Document all new features, improvements, and bug fixes included in `1.2.0`.
    * Commit and push:
        ```bash
        git add CHANGELOG.md
        git commit -m "docs: Update CHANGELOG for version 1.2.0"
        git push origin release/1.2.0
        ```
4.  **Obtain Final QA Sign-off:**
    * QA Team: Provide official approval for a specific Release Candidate build (e.g., `1.2.0+65`). Note this final `+BUILD_NUMBER`.

---

### Stage 3: Production Merge & Tagging

1.  **Create PR: `release/1.2.0` to `main`:**
    * Release Lead: Initiate the Pull Request.
    * **Action**: Team conducts a final, thorough review. Ensure all CI checks pass.
2.  **Merge PR to `main`:**
    * After approval, merge the PR.
3.  **Tag `main` for Release:**
    * After the merge is complete on `main`:
        ```bash
        git checkout main
        git pull origin main
        # Use the final QA-approved +BUILD_NUMBER in the tag message
        git tag -a v1.2.0 -m "Release version 1.2.0 (Build +65)"
        git push origin v1.2.0
        ```
4.  **Create PR: `release/1.2.0` to `develop`:**
    * Release Lead: Initiate Pull Request to merge `release/1.2.0` back into `develop`.
    * **Action**: Review and merge to ensure all fixes/updates are in `develop`.
5.  **(Optional) Delete `release` Branch:**
    ```bash
    git branch -d release/1.2.0
    git push origin --delete release/1.2.0
    ```

---

### Stage 4: Deployment & Post-Release

1.  **Build for Production:**
    * Generate the final production app package from the tagged commit (`v1.2.0`) on `main`. This build **must** use the final version name and build number (e.g., `1.2.0+65`).
2.  **Submit to App Stores:**
    * Upload the build to Google Play Console & App Store Connect.
    * Complete store listing information and initiate rollout.
3.  **Create GitHub Release:**
    * Navigate to your repository on GitHub > "Releases" > "Draft a new release".
    * **Choose tag:** `v1.2.0`.
    * **Release title:** `Version 1.2.0`.
    * **Description:** Copy relevant content from `CHANGELOG.md`.
    * Publish the release.
4.  **Monitor Production:**
    * Track crash reports, analytics, and user feedback closely.

---
---

## 3. Hotfix Procedure

**Use this procedure for:** Urgent fixes to critical bugs in the current production version.
**Example Scenario:** Production is `v1.2.0+65`. A critical bug needs immediate fixing. New hotfix version will be `1.2.1+66`.

---

### Stage 1: Hotfix Branch & Implementation

1.  **Identify Production Tag:**
    * Confirm the exact tag of the live production version (e.g., `v1.2.0`).
2.  **Create `hotfix` Branch from Production Tag on `main`:**
    ```bash
    git checkout main
    git pull origin main
    # Branch from the specific production tag (e.g., v1.2.0)
    git checkout -b hotfix/1.2.1-critical-login v1.2.0
    git push origin hotfix/1.2.1-critical-login
    ```
3.  **Implement the Fix on `hotfix/1.2.1-critical-login`:**
    * Apply code changes to resolve the critical bug.
    * Commit the fix: `fix: Critical - Description of fix`
4.  **Bump Version on `hotfix/1.2.1-critical-login`:**
    * Update app version to `1.2.1` (increment PATCH) and set new `+BUILD_NUMBER` (e.g., `+66`).
    * Modify files: `pubspec.yaml`, Android `build.gradle`, iOS `Info.plist`.
    * Commit and push: `chore: Bump version to 1.2.1+66 for hotfix`
5.  **Update `CHANGELOG.md` on `hotfix/1.2.1-critical-login`:**
    * Document the hotfix. Commit and push.
6.  **Thoroughly Test Hotfix Build:**
    * QA Team: Test builds from `hotfix/1.2.1-critical-login` (e.g., `1.2.1+66`). Focus on the fix and regression.
7.  **Obtain QA Sign-off for Hotfix:**
    * QA Team: Provide official approval for the hotfix build.

---

### Stage 2: Merge & Release Hotfix

1.  **Create PR: `hotfix/1.2.1-critical-login` to `main`:**
    * Initiate Pull Request.
    * **Action**: Conduct an urgent but meticulous review. Ensure CI passes.
2.  **Merge PR to `main`:**
    * After approval, merge the PR.
3.  **Tag `main` for Hotfix Release:**
    * After the merge is complete on `main`:
        ```bash
        git checkout main
        git pull origin main
        git tag -a v1.2.1 -m "Hotfix release version 1.2.1 (Build +66) - Fixes [brief issue description]"
        git push origin v1.2.1
        ```
4.  **Create PR: `hotfix/1.2.1-critical-login` to `develop`:**
    * Initiate Pull Request to merge the hotfix into `develop`.
    * **Action**: Review and merge.
5.  **Merge Hotfix into any Active `release/*` Branch (If Applicable):**
    * If a standard release (e.g., `release/1.3.0`) is in progress, create a PR from `hotfix/1.2.1-critical-login` to that `release/*` branch. Review and merge.
6.  **(Optional) Delete `hotfix` Branch:**
    ```bash
    git branch -d hotfix/1.2.1-critical-login
    git push origin --delete hotfix/1.2.1-critical-login
    ```

---

### Stage 3: Deploy & Monitor Hotfix

1.  **Build Hotfix for Production:**
    * Generate the final production app package from the tagged commit (`v1.2.1`) on `main`. This build will be `1.2.1+66`.
2.  **Submit to App Stores (Expedited):**
    * Upload the build. Request expedited review if available and necessary.
3.  **Create GitHub Release for Hotfix:**
    * Follow procedure in Standard Release (Stage 4, Step 3) for `v1.2.1`.
4.  **Monitor Production Closely:**
    * Track crash reports and feedback specific to the hotfix.

---
---

## 4. Appendix

### 4.1. Build Number (`+BUILD_NUMBER`)
* **MUST** be unique and greater than any previous build for that platform.
* **MUST** be incremented for every new build sent to QA from a `release/*` or `hotfix/*` branch, and for the final store submission.
* CI/CD can help automate incrementing.

### 4.2. Document Maintenance
* This SOP is a living document. Propose updates via PRs to this document as processes evolve.

