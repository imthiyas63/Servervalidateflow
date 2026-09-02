# Git Commit Message Validation Hooks

This repository uses **Git hooks** to validate commit messages before they are pushed to GitHub.

The validation ensures that every commit references a valid GitHub Issue and follows the required commit message format.

## Overview

The commit validation supports:

* Single-repository issue references
* Multiple issues in the same repository
* Issues from another repository in the same organization
* Issues from repositories in different organizations
* Local validation using Git hooks
* GitHub authentication using a Personal Access Token or GitHub CLI
* Server-side validation as a fallback when the local hook is not installed

---

# 1. Clone the Repository

Clone the repository and move into the repository directory.

```bash
git clone <repository-url>
cd <repository-directory>
```

Example:

```bash
git clone https://github.com/organization/gitrepo.git
cd gitrepo
```

---

# 2. Configure the Git Hooks

Configure Git to use the `.githooks` directory included in the repository.

```bash
git config core.hooksPath .githooks
```

Verify the configuration:

```bash
git config --get core.hooksPath
```

Expected output:

```text
.githooks
```

> **Important:** Run this configuration inside the repository so that the repository uses the validation hooks.

---

# 3. Authenticate with GitHub

The commit validation hook needs access to GitHub Issues to validate issue references.

Choose **ONE** of the following authentication methods.

## Option A: Personal Access Token

**Recommended for environments where GitHub CLI is not required.**

### Step 1: Generate a Fine-Grained Personal Access Token

Go to:

**GitHub.com → Settings → Developer settings → Personal access tokens → Fine-grained tokens**

Then:

1. Click **Generate new token**.

2. Give the token a descriptive name.

   Example:

   ```text
   Local Git Hooks
   ```

3. Set an expiration period such as:

   * 90 days
   * 1 year

4. Select the required repository access.

5. Under **Repository permissions**, grant the required read permissions for:

   * Issues
   * Contents

6. Click **Generate token**.

7. Copy the generated token.

> **Security:** Never commit the token to the repository or share it with other users.

### Step 2: Save the GitHub Credentials

Configure the token:

```bash
git config --global github.token "github_pat_YOUR_TOKEN_HERE"
```

Configure your GitHub username:

```bash
git config --global github.user "your-github-username"
```

Example:

```bash
git config --global github.token "github_pat_YOUR_TOKEN_HERE"
git config --global github.user "imthiyas63"
```

---

# 4. Option B: GitHub CLI

Instead of configuring a Personal Access Token manually, you can authenticate using the **GitHub CLI (`gh`)**.

## Step 1: Install GitHub CLI

On Windows:

```powershell
winget install --id GitHub.cli
```

After installation, refresh the `PATH`:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

Verify the installation:

```powershell
gh --version
```

## Step 2: Authenticate

Run:

```powershell
gh auth login
```

Select the following options:

```text
? Where do you use GitHub?
  GitHub.com

? What is your preferred protocol for Git operations on this host?
  HTTPS

? Authenticate Git with your GitHub credentials?
  Yes

? How would you like to authenticate GitHub CLI?
  Login with a web browser
```

GitHub CLI will display a one-time authentication code.

Example:

```text
First copy your one-time code: XXXX-XXXX
https://github.com/login/device
```

Open the displayed URL, enter the code, and complete the GitHub authentication.

After successful authentication, GitHub CLI will confirm that authentication was successful.

Verify authentication:

```powershell
gh auth status
```

---

# 5. Commit Message Format

Every commit must contain:

1. A valid GitHub Issue reference
2. A description
3. A description of at least **5 characters**
4. A valid separator between the issue reference and description

The supported formats depend on where the referenced Issue exists.

---

## 5.1 Single-Repo Projects

When the Issue belongs to the **same repository**, use:

```text
#<issue-number>: <description>
```

Examples:

```bash
git commit -m "#1: fixed login bug"
```

```bash
git commit -m "#123: added GST report feature"
```

```bash
git commit -m "#456 - updated dashboard UI"
```

Valid examples include:

```text
#1: fixed login bug
#123: added GST report feature
#456 - updated dashboard UI
#789. refactored database connection
```

---

# 5.2 Multi-Repo Projects

When the Issue belongs to **another repository within the same organization**, use:

```text
<repository-name>#<issue-number>: <description>
```

Examples:

```bash
git commit -m "gitrepo#1: fixed backend API"
```

```bash
git commit -m "ui-frontend#123: updated login page"
```

You can also reference multiple Issues:

```bash
git commit -m "#123 #456: refactored authentication and session management"
```

---

# 5.3 Cross-Organization Repositories

When the Issue belongs to a repository in the **same or a different GitHub organization**, use the complete repository reference:

```text
<organization>/<repository>#<issue-number>: <description>
```

Examples:

```bash
git commit -m "imthiyas63/gitrepo#1: cross-repo fix"
```

```bash
git commit -m "JavakarBits/ezeebus#456: backend changes"
```

```bash
git commit -m "imthiyas63/ui-frontend#123: UI updates"
```

This format explicitly identifies the GitHub repository containing the Issue.

---

# 6. Multiple Issue References

A commit can reference multiple Issues.

Example:

```bash
git commit -m "#123 #456: refactored authentication and session management"
```

Cross-repository references can also be used:

```bash
git commit -m "gitrepo#123 ui-frontend#456: updated authentication flow"
```

The commit description must still be meaningful and meet the minimum description length requirement.

---

# 7. Invalid Commit Formats

The following commit messages will be blocked by the validation hook.

## Missing Issue Number

❌ Invalid:

```bash
git commit -m "fixed bug"
```

Reason:

```text
No issue reference found
```

---

## Missing Description

❌ Invalid:

```bash
git commit -m "#1"
```

Reason:

```text
Missing description
```

---

## Description Too Short

❌ Invalid:

```bash
git commit -m "#1: fix"
```

Reason:

```text
Description too short (min 5 chars)
```

✅ Valid:

```bash
git commit -m "#1: fixed login validation bug"
```

---

## Wrong Separator

❌ Invalid:

```bash
git commit -m "#123fix bug"
```

The Issue reference and description must be separated using a supported separator.

---

## Missing `#` Symbol

❌ Invalid:

```bash
git commit -m "123: fixed bug"
```

The Issue number must contain the `#` symbol.

✅ Valid:

```bash
git commit -m "#123: fixed bug"
```

---

# 8. Common Issues and Fixes

## Error: `No issue reference found`

### Problem

Your commit message does not contain a GitHub Issue reference.

❌ Example:

```text
fixed login bug
```

### Fix

Amend the commit with a valid Issue reference:

```bash
git commit --amend -m "#123: fixed login validation bug"
```

Then push the amended commit:

```bash
git push --force origin <your-branch-name>
```

> Use `--force-with-lease` instead of `--force` when possible, especially on shared branches.

---

## Error: `Description too short (min 5 chars)`

### Problem

The commit description is too short.

❌ Bad:

```bash
git commit -m "#123: fix"
```

✅ Good:

```bash
git commit -m "#123: fixed login validation bug"
```

---

## Error: `Missing GitHub Token`

or:

```text
Cannot determine GitHub username
```

### Problem

The local hook cannot find the GitHub authentication information.

### Fix

Configure your GitHub token:

```bash
git config --global github.token "your_token_here"
```

Configure your GitHub username:

```bash
git config --global github.user "your_username"
```

Then verify:

```bash
git config --get github.user
```

```bash
git config --get github.token
```

---

# 9. GitHub CLI Authentication Problems

If you are using GitHub CLI, check your authentication status:

```powershell
gh auth status
```

If you are not authenticated:

```powershell
gh auth login
```

Complete the browser-based authentication process.

---

# 10. Error: `VALIDATION FAILED` on GitHub

### Problem

The commit worked locally, but GitHub rejected the push because the commit did not pass server-side validation.

This commonly happens when the local Git hook was not installed or configured correctly.

### Step 1: Configure the Local Hook

Run:

```bash
git config core.hooksPath .githooks
```

Verify:

```bash
git config --get core.hooksPath
```

Expected:

```text
.githooks
```

### Step 2: Fix the Commit

Amend the commit:

```bash
git commit --amend -m "#123: proper commit message"
```

Then push:

```bash
git push --force-with-lease origin <your-branch-name>
```

---

# 11. Verify Your Configuration

Use the following commands to verify the local setup.

## Check Hooks Path

```bash
git config --get core.hooksPath
```

Expected:

```text
.githooks
```

---

## Check GitHub Token

```bash
git config --get github.token
```

This should return the configured token.

> **Security:** Avoid displaying or sharing the token in screenshots, logs, terminals, or support requests.

If nothing is returned, the token is not configured through this Git configuration.

---

## Check GitHub Username

```bash
git config --get github.user
```

Expected output:

```text
your-github-username
```

---

## Check GitHub CLI Authentication

If you selected GitHub CLI authentication:

```powershell
gh auth status
```

---

# 12. Recommended Setup

For a new developer, the recommended setup is:

```bash
# Clone repository
git clone <repository-url>

# Enter repository
cd <repository-directory>

# Configure hooks
git config core.hooksPath .githooks

# Verify hooks
git config --get core.hooksPath
```

Then choose **one** authentication method:

### Personal Access Token

```bash
git config --global github.token "github_pat_YOUR_TOKEN_HERE"
git config --global github.user "your-github-username"
```

### OR GitHub CLI

```powershell
winget install --id GitHub.cli
gh auth login
gh auth status
```

Finally, create commits using the required format:

```bash
git commit -m "#123: fixed login validation bug"
```

---

# 13. Developer Workflow

The expected development workflow is:

```text
Clone Repository
       │
       ▼
Configure .githooks
       │
       ▼
Authenticate with GitHub
       │
       ▼
Create / Modify Code
       │
       ▼
git commit
       │
       ▼
Commit Message Validation
       │
       ├── ❌ Invalid
       │      │
       │      ▼
       │   Fix Commit Message
       │
       └── ✅ Valid
              │
              ▼
           git push
              │
              ▼
       Server-Side Validation
              │
              ├── ❌ Failed → Fix Commit
              │
              └── ✅ Passed
                     │
                     ▼
                  Continue
```

---

# 14. Quick Reference

| Purpose                   | Command                                       |
| ------------------------- | --------------------------------------------- |
| Clone repository          | `git clone <repository-url>`                  |
| Enter repository          | `cd <repository-directory>`                   |
| Configure hooks           | `git config core.hooksPath .githooks`         |
| Check hooks               | `git config --get core.hooksPath`             |
| Configure GitHub username | `git config --global github.user "username"`  |
| Configure GitHub token    | `git config --global github.token "token"`    |
| Check GitHub CLI          | `gh --version`                                |
| Login with GitHub CLI     | `gh auth login`                               |
| Check GitHub CLI auth     | `gh auth status`                              |
| Amend commit              | `git commit --amend -m "#123: description"`   |
| Push amended commit       | `git push --force-with-lease origin <branch>` |

---

# 15. Commit Format Quick Reference

### Same Repository

```text
#123: description
```

Example:

```bash
git commit -m "#123: fixed login validation bug"
```

### Another Repository in Same Organization

```text
repository#123: description
```

Example:

```bash
git commit -m "ui-frontend#123: updated login page"
```

### Different Organization / Explicit Repository

```text
organization/repository#123: description
```

Example:

```bash
git commit -m "imthiyas63/ui-frontend#123: UI updates"
```

### Multiple Issues

```text
#123 #456: description
```

Example:

```bash
git commit -m "#123 #456: refactored authentication and session management"
```

---

## Summary

Before committing code, make sure:

* [ ] Repository is cloned.
* [ ] `.githooks` is configured.
* [ ] `git config --get core.hooksPath` returns `.githooks`.
* [ ] GitHub authentication is configured.
* [ ] Every commit references a GitHub Issue.
* [ ] Every commit contains a meaningful description.
* [ ] The description contains at least 5 characters.
* [ ] Cross-repository Issues use the appropriate repository format.
* [ ] Commits pass local validation before pushing.
* [ ] Commits also pass server-side validation on GitHub.

