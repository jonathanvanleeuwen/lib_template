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

# 🔒 Secure GitHub Repository Setup

This section covers how to configure your GitHub repository for secure CI/CD operations. **Follow these steps in order.**

> ⚠️ **Important:** Some settings (like status checks) require the workflows to have run at least once before they appear as options. This guide includes a step to trigger the workflows first.

## Step 1: Create the Release Environment

The workflows use a `release` environment that requires approval before running critical operations (pushing to main, creating releases).

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Environments**
3. Click **New environment**
4. Name it exactly: `release`
5. Click **Configure environment**
6. Under **Environment protection rules**:
   - ✅ **Required reviewers** → Add yourself (and any trusted collaborators who should be able to approve)

> **Note:** If you add collaborators as reviewers, they can approve deployments without needing YOUR explicit approval each time. This is useful for teams where multiple people should be able to release.

7. Click **Save protection rules**

---

## Step 2: Configure Actions Permissions

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

## Step 3: Create Classic Branch Protection Rule (Partial)

We use **classic branch protection** (not rulesets) because it properly supports `github-actions[bot]` bypass.

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
| ✅ **Do not allow bypassing the above settings** | |
| ✅ **Restrict who can push to matching branches** | See below ↓ |

### Critical: Allow GitHub Actions to Push

4. Under **Restrict who can push to matching branches**:
   - Click the search box
   - Type: `github-actions`
   - Select: **github-actions[bot]** (it appears as a GitHub App)

This restricts direct pushes to ONLY `github-actions[bot]`, while all humans must use PRs.

> **Note:** We're NOT enabling "Require status checks to pass" yet — we'll add that after triggering the workflows.

5. Click **Create** (or **Save changes**)

---

## Step 4: Trigger Workflows to Register Status Checks

Status checks only appear in GitHub's dropdown **after the workflows have run at least once**. Let's trigger them:

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

5. Wait for the workflows to run (you'll see them in the PR's "Checks" section)
   - `Run Pre-commit Checks` should appear
   - `Run Tests and Lint` should appear

6. Once checks pass, **merge the PR** (you may need to approve it first since you're the code owner)

7. After merging, the release pipeline will trigger. Go to **Actions** tab and **approve the deployment** to the `release` environment when prompted.

---

## Step 5: Add Status Checks to Branch Protection

Now that the workflows have run, we can add the status checks:

1. Go to **Settings** → **Branches**
2. Click **Edit** on your `main` branch protection rule

3. Enable: ✅ **Require status checks to pass before merging**
4. Enable: ✅ **Require branches to be up to date before merging**
5. In the search box, search for and add:
   - `Run Tests and Lint`
   - `Run Pre-commit Checks`

> **Note:** When you search, you'll see the check names. Select them to add as required checks.

6. Click **Save changes**

---

## 🔐 Security Summary

| Actor | Can push to main | Can change workflows | Can trigger release |
|-------|------------------|---------------------|---------------------|
| **You (owner)** | ✅ via PR only | ✅ via PR only | ✅ (can approve) |
| **GitHub Actions** | ✅ directly (bypass) | ❌ (CODEOWNERS blocks) | ✅ (after environment approval) |
| **Collaborators** | ❌ (unless PR approved) | ❌ (unless you approve) | ✅ if added as environment reviewer |
| **Fork / Public** | ❌ | ❌ | ❌ |

### About Environment Approval:

- **You** can add collaborators as "Required reviewers" in the `release` environment
- Any reviewer can approve the deployment — it doesn't require ALL reviewers
- This means trusted collaborators can approve releases without waiting for you
- To add collaborators: **Settings** → **Environments** → `release` → **Required reviewers**

### How It Works:

1. **CODEOWNERS file** (included in template) requires YOUR approval for any changes to `.github/workflows/`
2. **Classic branch protection** allows only `github-actions[bot]` to push directly to main
3. **Environment protection** requires approval before release jobs run
4. **Result:** Workflows can push (coverage, versions, wheels), but no one can modify the workflows without your review

### What We Removed (Insecure):
- ❌ Personal Access Tokens (PATs) - can bypass ALL protections
- ❌ `actions-js/push@master` - replaced with native git commands
- ❌ Mutable action tags (`@master`) - security risk

### What We Added (Secure):
- ✅ `GITHUB_TOKEN` with limited permissions
- ✅ Environment protection with required reviewers
- ✅ CODEOWNERS file protecting workflow files
- ✅ Explicit `permissions:` blocks in workflows
- ✅ Classic branch protection allowing only Actions to push directly

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
