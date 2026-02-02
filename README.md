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
* **Run pre-commit on all files** (important for initial commit!)
  * `pre-commit run --all-files`
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

# 🔒 GitHub Repository Setup

This section covers how to configure your GitHub repository for CI/CD operations. **Follow these steps in order.**

> ⚠️ **Important:** Status checks only appear after workflows have run at least once. This guide includes a step to trigger workflows first.

## Step 1: Configure Actions Permissions

1. Go to **Settings** → **Actions** → **General**

2. Under **Actions permissions**:
   - Select: ✅ **Allow all actions and reusable workflows**

3. Under **Workflow permissions**:
   - Select: ✅ **Read and write permissions**
   - Enable: ✅ **Allow GitHub Actions to create and approve pull requests**

4. Under **Fork pull request workflows from outside collaborators** (for public repos):
   - Select: ✅ **Require approval for first-time contributors**

5. Click **Save**

---

## Step 2: Create Branch Protection Rule

1. Go to **Settings** → **Branches**
2. Click **Add branch protection rule** (or **Add classic branch protection rule**)
3. Set **Branch name pattern**: `main`

### Configure these settings:

| Setting | Value |
|---------|-------|
| ✅ **Require a pull request before merging** | |
|     | → Required approvals: `1` (or more) |
|     | → ✅ Dismiss stale pull request approvals when new commits are pushed |
|     | → ✅ Require review from Code Owners |
| ✅ **Require conversation resolution before merging** | |
| ❌ **Do not allow bypassing the above settings** | **UNCHECKED** (allows admin bypass) |

> **Note:** Leave "Do not allow bypassing the above settings" **UNCHECKED** so you (as admin) can bypass if needed, and so GitHub Actions can push commits.

4. Click **Create**

> **Note:** We're NOT enabling "Require status checks to pass" yet — we'll add that after triggering the workflows in Step 3.

---

## Step 3: Trigger Workflows to Register Status Checks

Status checks only appear in GitHub's dropdown **after the workflows have run at least once**.

1. Create a new branch locally:
   ```bash
   git checkout -b test/trigger-workflows
   ```

2. Make a small change (e.g., add a comment to any file)

3. Commit and push:
   ```bash
   git add .
   git commit -m "fix: trigger initial workflow run"
   git push -u origin test/trigger-workflows
   ```

4. Go to GitHub and **create a Pull Request** from `test/trigger-workflows` → `main`

5. Wait for the workflows to run (you'll see them in the PR's "Checks" section):
   - `Run Pre-commit Checks`
   - `Run Tests and Lint`

6. Once checks pass, **merge the PR** (approve it since you're the code owner)

7. After merging, the release pipeline runs automatically. Check the **Actions** tab to verify it completes successfully.

---

## Step 4: Add Status Checks to Branch Protection

Now that the workflows have run, add the status checks:

1. Go to **Settings** → **Branches**
2. Click **Edit** on your `main` branch protection rule

3. Enable: ✅ **Require status checks to pass before merging**
4. Enable: ✅ **Require branches to be up to date before merging**
5. In the search box, search for and add:
   - `Run Tests and Lint`
   - `Run Pre-commit Checks`

6. Click **Save changes**

---

## 🔐 Security Summary

| Actor | Can push to main | Can approve PRs | Notes |
|-------|------------------|-----------------|-------|
| **You (admin)** | ✅ can bypass | ✅ | Admin bypass enabled |
| **GitHub Actions** | ✅ directly | N/A | Pushes coverage, versions, wheels |
| **Collaborators** | ❌ PR required | ✅ if reviewer | Standard PR workflow |
| **Fork / Public** | ❌ | ❌ | Cannot push |

### How the Protection Works:

1. **PRs required for humans** - All collaborators must go through PRs
2. **Admin bypass available** - You can push directly if needed (emergency fixes)
3. **GitHub Actions can push** - Workflows push coverage reports, version bumps, and wheels automatically
4. **CODEOWNERS protection** - Changes to `.github/workflows/` require your approval
5. **Status checks** - Tests and linting must pass before merging

### What Changed from PAT-based Setup:

| Before (PAT) | After (GITHUB_TOKEN) |
|--------------|---------------------|
| ❌ PAT could bypass ALL protections | ✅ Only Actions can push automatically |
| ❌ PAT in secrets = security risk | ✅ GITHUB_TOKEN is built-in and scoped |
| ❌ PAT never expires (unless set) | ✅ GITHUB_TOKEN expires per-job |

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
