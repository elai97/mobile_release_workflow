# Mobile App Release Workflow & Versioning Guide

**Document Version:** 1.0
**Last Updated:** May 22, 2025

## 1. Introduction

This document outlines the standardized release workflow and versioning conventions for our mobile application. Adhering to this process will help ensure smooth, predictable, and high-quality releases.

### 1.1. Versioning Convention

We use a modified Semantic Versioning scheme combined with a build number:

**Format:** `MAJOR.MINOR.PATCH+BUILD_NUMBER`
* **`MAJOR`** (e.g., `1`.x.x): Incremented for significant new features or breaking changes that might affect the core user experience or require data migration.
* **`MINOR`** (e.g., x.`1`.x): Incremented when new, backward-compatible functionality is added.
* **`PATCH`** (e.g., x.x.`7`): Incremented for backward-compatible bug fixes.
* **`+BUILD_NUMBER`** (e.g., `+73`): A unique, always-incrementing integer for every build submitted to app stores or distributed for testing.
    * **Android:** This corresponds to `versionCode`.
    * **iOS:** This corresponds to `CFBundleVersion` (Build).

**Example:** `1.0.7+73` (Version Name: 1.0.7, Build Number: 73)

### 1.2. Branching Strategy Overview

We follow a GitFlow-inspired model:
* **`main`**: Production-released code. Always stable.
* **`develop`**: Main integration branch for ongoing development.
* **`feature/<feature-name>`**: For developing new features.
* **`release/<version-name>`**: For preparing a release (e.g., `release/1.2.0`).
* **`hotfix/<fix-description-or-version>`**: For urgent production fixes.

### 1.3. Core Principles
* **Pull Requests (PRs)**: All merges into `develop` and `main` (and `release/*`, `hotfix/*`) must be done via PRs.
* **Code Reviews**: PRs require at least one approval from another team member.
* **CI/CD**: Automated tests and builds should be integrated into the workflow.
* **Changelog**: Maintain a `CHANGELOG.md`.

---

## 2. Standard Release Workflow (Minor/Major/Patch)

This process is for planned releases that include new features, improvements, or non-critical bug fixes. Let's assume we are releasing version `1.2.0`. The current production version is `1.1.5+60`.

**Key Roles:**
* **Release Lead:** Person responsible for coordinating the release.
* **Development Team:** Developers contributing features and fixes.
* **QA Team:** Testers verifying the release candidate.

---

**Phase 1: Feature Development & Integration**

1.  **Feature Branches (`feature/*`)**:
    * Developers create feature branches from `develop`:
        ```bash
        git checkout develop
        git pull origin develop
        git checkout -b feature/new-user-profile
        ```
    * Develop features, commit changes.
    * Regularly sync with `develop`:
        ```bash
        git pull origin develop # Or git rebase develop
        ```
2.  **Merge Features to `develop`**:
    * Once a feature is complete and tested locally, create a PR from `feature/new-user-profile` to `develop`.
    * After code review and CI checks pass, merge the PR.
    * Delete the feature branch.

---

**Phase 2: Release Preparation (Example: Releasing `1.2.0`)**

1.  **Feature Freeze on `develop`**:
    * The Release Lead announces a feature freeze for version `1.2.0`. No new features are merged into `develop` for this release. Only bug fixes relevant to the upcoming release are allowed.
2.  **Create `release` Branch**:
    * The Release Lead (or designated person) creates the release branch from `develop`:
        ```bash
        git checkout develop
        git pull origin develop
        git checkout -b release/1.2.0
        git push origin release/1.2.0
        ```
    * From this point, `develop` is open for features intended for *future* releases (e.g., `1.3.0`).
3.  **On the `release/1.2.0` Branch**:
    * **Bump Version Number**:
        * Update the version name to `1.2.0` and assign a new `BUILD_NUMBER` (e.g., `+61` if the last build was `+60`).
        * Modify `pubspec.yaml` (Flutter), `build.gradle` (Android), `Info.plist` (iOS).
        * Example commit message: `chore: Bump version to 1.2.0+61 for release`
        ```bash
        # Example: git commit -am "chore: Bump version to 1.2.0+61 for release"
        # git push origin release/1.2.0
        ```
    * **Stabilization & QA**:
        * QA team tests builds from the `release/1.2.0` branch.
        * Any bugs found are fixed directly on `release/1.2.0`. Each fix should be a separate commit.
        * Example commit message: `fix: Resolve crash on settings screen (release/1.2.0)`
    * **Update `CHANGELOG.md`**:
        * Document all new features, improvements, and bug fixes included in this release.
        * Commit the updated changelog.
        * Example commit message: `docs: Update CHANGELOG for version 1.2.0`
    * **Build Release Candidates (RCs)**:
        * Generate builds (e.g., `1.2.0+61`, `1.2.0+62` if fixes are made) from `release/1.2.0` for internal testing, TestFlight, Google Play Internal/Closed Testing, or Firebase App Distribution.
        * Each RC build should have an incremented `BUILD_NUMBER`.

---

**Phase 3: Production Release**

1.  **Final Approval**:
    * Once QA confirms the release candidate (e.g., `1.2.0+65`) is stable and ready for production.
2.  **Merge `release/1.2.0` to `main`**:
    * The Release Lead creates a PR from `release/1.2.0` to `main`.
    * **Crucial Review**: This PR should be carefully reviewed, as `main` represents production code.
    * After approval and passing CI checks, merge the PR.
        ```bash
        # (Ensure local main is up-to-date after PR merge via GitHub UI)
        git checkout main
        git pull origin main
        ```
3.  **Tag `main`**:
    * Create an annotated tag on `main` for the exact release commit. The tag should match the version name.
        ```bash
        git tag -a v1.2.0 -m "Release version 1.2.0 (Build +65)"
        git push origin v1.2.0 # Push the specific tag
        # Or git push origin --tags # Push all local tags (use with caution)
        ```
    * The build number in the tag message (`+65`) is the final build number that will be submitted to the stores.
4.  **Merge `release/1.2.0` back to `develop`**:
    * This ensures any fixes or version bumps made on the `release` branch are incorporated back into `develop`.
    * Create a PR from `release/1.2.0` to `develop`.
    * Review and merge.
        ```bash
        # (Ensure local develop is up-to-date after PR merge via GitHub UI)
        git checkout develop
        git pull origin develop
        ```
5.  **(Optional) Delete `release` Branch**:
    * Once merged to `main` and `develop`, the `release/1.2.0` branch can be deleted.
        ```bash
        git branch -d release/1.2.0
        git push origin --delete release/1.2.0
        ```

---

**Phase 4: Store Submission & Post-Release**

1.  **Build for Production**:
    * Generate the final production build from the tagged commit (`v1.2.0`) on the `main` branch. This build will have the version `1.2.0+65`.
2.  **Submit to App Stores**:
    * Upload the build to Google Play Console and App Store Connect.
    * Complete store listing information, rollout percentages, etc.
3.  **Monitor Release**:
    * Track crash reports (Firebase Crashlytics, Sentry), analytics, and user feedback.
4.  **GitHub Release**:
    * Navigate to your repository on GitHub.
    * Go to "Releases" and "Draft a new release".
    * Choose the tag you created (e.g., `v1.2.0`).
    * Title the release (e.g., `Version 1.2.0`).
    * Copy content from `CHANGELOG.md` into the release description.
    * (Optional) Attach build artifacts (e.g., `.apk`, `.ipa` if not directly submitted from CI).
    * Publish the release.

---

## 3. Hotfix Release Workflow

This process is for addressing critical bugs in a live production version that cannot wait for the next standard release. Let's assume production is `v1.2.0+65` and a critical bug is found. We need to release `v1.2.1`.

1.  **Create `hotfix` Branch from `main`**:
    * Branch from the *tag* of the version you are hotfixing on `main`.
        ```bash
        git checkout main
        git pull origin main # Ensure main is up-to-date
        git checkout -b hotfix/1.2.1-critical-login v1.2.0 # Branch from the v1.2.0 tag
        git push origin hotfix/1.2.1-critical-login
        ```
2.  **On the `hotfix/1.2.1-critical-login` Branch**:
    * **Implement the Fix**: Make the necessary code changes to fix the critical bug. Commit the fix.
        * Example commit message: `fix: Critical - Resolve login failure for existing users`
    * **Bump Version Number**:
        * Update the version name to `1.2.1` (increment PATCH).
        * Assign a new, incremented `BUILD_NUMBER` (e.g., `+66`).
        * Modify `pubspec.yaml`, `build.gradle`, `Info.plist`.
        * Example commit message: `chore: Bump version to 1.2.1+66 for hotfix`
    * **Thorough Testing**: Test this specific fix extensively.
    * **Update `CHANGELOG.md`**: Document the hotfix.
3.  **Merge `hotfix` to `main`**:
    * Create a PR from `hotfix/1.2.1-critical-login` to `main`.
    * **Urgent & Careful Review**: This is critical. Review thoroughly.
    * After approval and CI, merge the PR.
        ```bash
        git checkout main
        git pull origin main
        ```
4.  **Tag `main` for Hotfix Release**:
    * Tag the new commit on `main`.
        ```bash
        git tag -a v1.2.1 -m "Hotfix release version 1.2.1 (Build +66) - Fixes critical login issue"
        git push origin v1.2.1
        ```
5.  **Merge `hotfix` back to `develop`**:
    * Ensure the fix is also in the main development line.
    * Create a PR from `hotfix/1.2.1-critical-login` to `develop`. Review and merge.
        ```bash
        git checkout develop
        git pull origin develop
        ```
6.  **Merge `hotfix` into Active `release/*` Branch (if any)**:
    * If there's an active `release/*` branch (e.g., `release/1.3.0` was in progress), the hotfix also needs to be merged into it to prevent regression. Create a PR from `hotfix/1.2.1-critical-login` to `release/active-release-branch`.
7.  **(Optional) Delete `hotfix` Branch**:
    ```bash
    git branch -d hotfix/1.2.1-critical-login
    git push origin --delete hotfix/1.2.1-critical-login
    ```
8.  **Deploy Hotfix**:
    * Build from the tagged commit (`v1.2.1`) on `main`.
    * Submit to app stores (often as an expedited release if the platform supports it).
    * Create a GitHub Release for `v1.2.1`.

---

## 4. Appendix: Build Number Management

* The `BUILD_NUMBER` **must** be unique and higher than any previously submitted build number for that platform.
* **Strategy for `BUILD_NUMBER`**:
    * **Simple Increment**: The easiest is to simply increment it by 1 for every build intended for testing or store submission (e.g., `+61`, `+62`, `+63`).
    * **CI/CD Driven**: Your CI/CD system can often manage this, pulling the latest build number from the store or a central counter and incrementing it.
* Ensure your build scripts correctly pick up and apply the version name and build number to the app artifacts.

---

This document serves as a living guide. Please suggest improvements or clarifications as our team and processes evolve.
