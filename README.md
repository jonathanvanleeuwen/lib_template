# lib_template
A cookiecutter template for Python libraries with modern CI/CD setup.

## Features
* Automated testing on PR using GitHub Actions
* Pre-commit hooks for code quality (ruff, isort, trailing whitespace, etc.)
* Semantic release using GitHub Actions
* Automatic code coverage report in README
* Automatic wheel build and GitHub Release publishing
* Modern Python packaging with pyproject.toml

*Notes*
Workflows trigger when a branch is merged into main!
To install, please follow all the instructions in this readme.

**Security:** This template uses `GITHUB_TOKEN` with branch protection bypass instead of Personal Access Tokens (PATs). This is more secure because:
- PATs can bypass ALL protections and be abused if workflow files are modified
- `GITHUB_TOKEN` with proper branch protection only allows Actions to push, while humans must follow PR rules
- Environment protection gates add an extra approval layer before critical operations

If you followed all the steps, whenever a PR is merged into `main`, the workflows are triggered and should:
* Run pre-commit checks (fail fast on code quality issues)
* Ensure that tests pass (before merge)
* Create a code coverage report and commit that to the bottom of the README
* Create a semantic release (if you follow the semantic release pattern) and automatically update the version number of your code
* Build a wheel and publish it as a GitHub Release asset


# Install
Cookiecutter template:
* Cd to your new library location
  * `cd /your/new/library/path/`
* Install cookiecutter using pip
  * `pip install cookiecutter`
* Run the cookiecutter template from this GitHub repo
  * `cookiecutter https://github.com/jonathanvanleeuwen/lib_template`
* Fill in your new library values (including your GitHub username for CODEOWNERS)
* Create new virtual environment
  *  `python -m venv .venv`
* Activate the environment and install library with dev dependencies
  *  `pip install -e ".[dev]"`
* Install pre-commit hooks
  * `pip install pre-commit`
  * `pre-commit install`
* Check proper install by running tests
  * `pytest`

## Turn the new local cookiecutter code into a git repo

Open git bash
```bash
cd C:/your/code/directory
```
To init the repository, add all files and commit
```bash
git init
git add *
git add .github
git add .gitignore
git add .pre-commit-config.yaml
git commit -m "fix: Initial commit"
```

To add the new git repository to your GitHub:
*  Go to [github](https://github.com/).
-  Log in to your account.
-  Click the [new repository](https://github.com/new) button in the top-right. You'll have an option there to initialize the repository with a README file, but don't. Leave the repo empty
- Give the new repo the same name you gave your repo with the cookiecutter
-  Click the "Create repository" button.

Now we want to make sure we are using `main` as main branch name and push the code to GitHub
```bash
git remote add origin https://github.com/username/new_repo_name.git
git branch -M main
git push -u origin main
```

---

# 🔒 Secure GitHub Repository Setup

This section covers how to configure your GitHub repository for secure CI/CD operations. **Follow these steps in order.**

## Step 1: Create the Release Environment

The workflows use a `release` environment that requires approval before running critical operations (pushing to main, creating releases).

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Environments**
3. Click **New environment**
4. Name it exactly: `release`
5. Click **Configure environment**
6. Under **Environment protection rules**, enable:
   - ✅ **Required reviewers** → Add yourself (or trusted collaborators)
7. Click **Save protection rules**

> **Why?** Even if someone modifies a workflow to abuse bypass rights, the job won't run without your approval.

---

## Step 2: Configure Branch Protection for `main`

1. Go to **Settings** → **Branches**
2. Click **Add branch ruleset** (or **Add rule** for classic protection)
3. Set **Branch name pattern**: `main`
4. Enable the following protections:

### Required Settings:
| Setting | Value |
|---------|-------|
| **Require a pull request before merging** | ✅ Enabled |
| → Require approvals | Set to 1 (or more) |
| → Dismiss stale PR approvals when new commits are pushed | ✅ Enabled |
| **Require status checks to pass before merging** | ✅ Enabled |
| → Require branches to be up to date before merging | ✅ Enabled |
| → Add status checks: `Run Tests and Lint`, `Run Pre-commit Checks` | ✅ Add these |
| **Require conversation resolution before merging** | ✅ Enabled |
| **Do not allow bypassing the above settings** | ✅ Enabled |

### Critical Security Setting:
| Setting | Value |
|---------|-------|
| **Allow specified actors to bypass required pull requests** | ✅ Enabled |
| → Add: `github-actions[bot]` | ✅ Add this actor |

> **Why?** This allows your GitHub Actions workflows to push directly to `main` (for coverage updates, version bumps, wheel commits), while **humans must always go through PRs**.

5. Click **Create** or **Save changes**

---

## Step 3: Configure Repository Actions Permissions

1. Go to **Settings** → **Actions** → **General**
2. Under **Actions permissions**, select:
   - ✅ **Allow all actions and reusable workflows**
   - (Or for extra security: **Allow select actions** and add the specific actions used)

3. Under **Workflow permissions**:
   - Select: ✅ **Read and write permissions**
   - Enable: ✅ **Allow GitHub Actions to create and approve pull requests**

4. Click **Save**

> **Why?** The workflows need write access to push commits (coverage, wheel, version bumps). The "create and approve PRs" option is needed for some automation scenarios.

---

## Step 4: Enable CODEOWNERS Protection

The template includes a `.github/CODEOWNERS` file that requires your approval for changes to:
- `.github/workflows/**` (workflow files)
- `.github/CODEOWNERS` (the CODEOWNERS file itself)
- `pyproject.toml` (package configuration)

**To make CODEOWNERS work:**

1. Go to **Settings** → **Branches** → Edit your `main` branch rule
2. Enable: ✅ **Require review from Code Owners**
3. Save changes

> **Why?** This prevents attackers from modifying workflow files to abuse the GITHUB_TOKEN bypass rights. They'd need YOUR approval to change any workflow.

---

## Step 5: Restrict Fork Pull Request Workflows (Public Repos)

If your repository is **public**, add extra protection:

1. Go to **Settings** → **Actions** → **General**
2. Under **Fork pull request workflows from outside collaborators**:
   - Select: ✅ **Require approval for first-time contributors**
   - Or more strict: **Require approval for all outside collaborators**

> **Why?** Prevents malicious forks from running workflows that could abuse repository access.

---

## 🔐 Security Summary

| Actor | Can push to main | Can change workflows | Can trigger release |
|-------|------------------|---------------------|---------------------|
| **You (owner)** | ✅ via PR only | ✅ via PR only | ✅ after approval |
| **GitHub Actions** | ✅ directly | ❌ | ✅ after environment approval |
| **Collaborators** | ❌ (unless PR approved) | ❌ (unless you approve) | ❌ |
| **Fork / Public** | ❌ | ❌ | ❌ |

### What We Removed (Insecure):
- ❌ Personal Access Tokens (PATs) - can bypass ALL protections
- ❌ `actions-js/push@master` - replaced with native git commands
- ❌ Mutable action tags (`@master`) - security risk

### What We Added (Secure):
- ✅ `GITHUB_TOKEN` with limited permissions
- ✅ Environment protection with required reviewers
- ✅ CODEOWNERS file protecting workflow files
- ✅ Explicit `permissions:` blocks in workflows
- ✅ Branch protection allowing only Actions to bypass

---

---

# Semantic Release
https://python-semantic-release.readthedocs.io/en/latest/

The workflows are triggered when you merge into main!

When committing use the following format for your commit message:
* **patch** (0.0.X):
  ```
  fix: commit message
  ```
* **minor** (0.X.0):
  ```
  feat: commit message
  ```
* **major/breaking** (X.0.0) - add the breaking change on the third line:
  ```
  feat: commit message

  BREAKING CHANGE: description of breaking change
  ```

---

# Installation Options (for generated libraries)
Libraries created with this template support multiple installation methods:
Note that the @v1.0.0 can be updated for a specific version, or removed for the newest version

### Configure git credentials (more secure, recommended)
This method doesn't expose your token in command history:

```bash
# Store credentials in git (one-time setup)
git config --global credential.helper store

# Then install normally - git will prompt for credentials once
pip install "git+https://github.com/username/repo_name.git@v1.1.0"
# When prompted: username = your GitHub username, password = your PAT
```

### Option 1: Install from Private GitHub Release (Recommended)
```bash
# Using pip with a personal access token
pip install "git+https://YOUR_TOKEN@github.com/username/repo_name.git@v1.0.0"

# Using uv
uv pip install "git+https://YOUR_TOKEN@github.com/username/repo_name.git@v1.0.0"
```

### Option 2: Install from Wheel File in Repository
After cloning, install the wheel file from the `dist/` directory:
```bash
pip install repo_name/dist/repo_name-1.0.0-py3-none-any.whl
```

### Option 3: Install from Source (Clone Repository)
```bash
git clone https://github.com/username/repo_name.git
cd repo_name
pip install -e ".[dev]"
```

### Option4: Building a Wheel File Locally
```bash
pip install build
python -m build --wheel
# The wheel will be created in the dist/ directory
```
### Option 5: Add to requirements.txt or pyproject.toml

**In requirements.txt:**

```txt
# Using git+https (requires GH_TOKEN environment variable to be set)
repo_name @ git+https://github.com/username/repo_name.git@v1.1.0
```

**In pyproject.toml (for projects using PEP 621):**

```toml
[project]
dependencies = [
    "repo_name @ git+https://github.com/username/repo_name.git@v1.1.0",
]
```

> **Note:** When installing from requirements.txt with a private repo, ensure your git credentials are configured (see Option 1, Step 2, Option B above).


# Coverage report
<!-- Pytest Coverage Comment:Begin -->
<!-- Pytest Coverage Comment:End -->
