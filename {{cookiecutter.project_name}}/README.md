# {{cookiecutter.project_name}}
A Python library with modern CI/CD setup.

## Features
* Automated testing on PR using GitHub Actions
* Pre-commit hooks for code quality (ruff, isort, trailing whitespace, etc.)
* Semantic release using GitHub Actions
* Automatic code coverage report in README
* Automatic wheel build and GitHub Release publishing
* Modern Python packaging with pyproject.toml

*Notes*
Workflows trigger when a branch is merged into main!

**Security:** This project uses `GITHUB_TOKEN` with branch protection bypass instead of Personal Access Tokens (PATs). See the setup instructions below.

If you followed all the setup steps, whenever a PR is merged into `main`, the workflows will:
* Run pre-commit checks (fail fast on code quality issues)
* Ensure that tests pass (before merge)
* Create a code coverage report and commit that to the bottom of the README
* Create a semantic release (if you follow the semantic release pattern) and automatically update the version number
* Build a wheel and publish it as a GitHub Release asset


# Installation

## Option 1: Install from Private GitHub Release (Recommended)
Since this is a private repository, you need to authenticate with a GitHub Personal Access Token (PAT).

### Configure git credentials (more secure, recommended)
This method doesn't expose your token in command history:

```bash
# Store credentials in git (one-time setup)
git config --global credential.helper store

# Then install normally - git will prompt for credentials once
pip install "git+https://github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git@VERSION"
# When prompted: username = your GitHub username, password = your PAT
```

### Step 1: Create a Personal Access Token (one-time setup)

1. Go to [GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)](https://github.com/settings/tokens)
2. Click **"Generate new token (classic)"**
3. Give it a descriptive name (e.g., `{{cookiecutter.project_name}}-install`)
4. Select the **`repo`** scope (required for private repositories)
5. Click **"Generate token"**
6. **Copy the token immediately** - you won't be able to see it again!

### Step 2: Install the package

```bash
# Replace YOUR_TOKEN with your actual token and VERSION with the desired version (e.g., v1.0.0)
pip install "git+https://YOUR_TOKEN@github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git@VERSION"

# Install the latest version (main branch):
pip install "git+https://YOUR_TOKEN@github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git"
```

### Using uv (faster alternative to pip)

```bash
uv pip install "git+https://YOUR_TOKEN@github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git@v1.0.0"
```

## Option 2: Install from Wheel File in Repository

The latest wheel files are also committed to the `dist/` directory in the repository. After cloning:

```bash
# Clone the repository first
git clone https://github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git

# Install the wheel file directly
pip install {{cookiecutter.project_name}}/dist/{{cookiecutter.project_name}}-1.0.0-py3-none-any.whl
```

> **Note:** Replace the version number with the actual version in the `dist/` directory.

## Option 3: Install from Source (Clone Repository)

```bash
# Clone the repository
git clone https://github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git
cd {{cookiecutter.project_name}}

# Install using pip
pip install .

# Or install in editable/development mode with dev dependencies
pip install -e ".[dev]"
```

## Option 4: Add to requirements.txt or pyproject.toml

**In requirements.txt:**

```txt
{{cookiecutter.project_name}} @ git+https://github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git@v1.0.0
```

**In pyproject.toml (for projects using PEP 621):**

```toml
[project]
dependencies = [
    "{{cookiecutter.project_name}} @ git+https://github.com/{{cookiecutter.github_username}}/{{cookiecutter.project_name}}.git@v1.0.0",
]
```

## Building a Wheel File Locally

```bash
pip install build
python -m build --wheel
# The wheel will be created in the dist/ directory
```


# Development Setup

1. Create new virtual environment
   ```bash
   python -m venv .venv
   ```
2. Activate the environment and install library with dev dependencies
   ```bash
   pip install -e ".[dev]"
   ```
3. Install pre-commit hooks
   ```bash
   pip install pre-commit
   pre-commit install
   ```
4. **Run pre-commit on all files** (important!)
   ```bash
   pre-commit run --all-files
   ```
5. Check proper install by running tests
   ```bash
   pytest
   ```

---

# 🔒 GitHub Repository Setup

Follow these steps **in order** to configure your GitHub repository for CI/CD.

> ⚠️ **Important:** Status checks only appear after workflows have run once. See Step 3.

## Step 1: Configure Actions Permissions

1. Go to **Settings** → **Actions** → **General**
2. **Actions permissions**: ✅ **Allow all actions and reusable workflows**
3. **Workflow permissions**:
   - ✅ **Read and write permissions**
   - ✅ **Allow GitHub Actions to create and approve pull requests**
4. **Fork pull request workflows**: ✅ **Require approval for first-time contributors**
5. Click **Save**

## Step 2: Create Branch Protection Rule

1. Go to **Settings** → **Branches**
2. Click **Add branch protection rule**
3. Set **Branch name pattern**: `main`

| Setting | Value |
|---------|-------|
| ✅ **Require a pull request before merging** | |
|     | → Required approvals: `1` |
|     | → ✅ Dismiss stale pull request approvals when new commits are pushed |
|     | → ✅ Require review from Code Owners |
| ✅ **Require conversation resolution before merging** | |
| ❌ **Do not allow bypassing the above settings** | **UNCHECKED** (allows admin bypass) |

4. Click **Create** (we'll add status checks after Step 3)

## Step 3: Trigger Workflows & Add Status Checks

1. Create a branch, make a small change, push, and open a PR
2. Wait for workflows to run (`Run Tests and Lint`, `Run Pre-commit Checks`)
3. Merge the PR → Release pipeline runs automatically
4. Go back to **Settings** → **Branches** → Edit the `main` rule
5. Enable: ✅ **Require status checks to pass before merging**
   - ✅ **Require branches to be up to date before merging**
   - Search & add: `Run Tests and Lint`, `Run Pre-commit Checks`
6. Click **Save changes**

---

# Semantic Release
https://python-semantic-release.readthedocs.io/en/latest/

Workflows are triggered when you merge into main!

Commit message formats:
* **patch** (0.0.X): `fix: commit message`
* **minor** (0.X.0): `feat: commit message`
* **major** (X.0.0):
  ```
  feat: commit message

  BREAKING CHANGE: description
  ```

# Coverage report
<!-- Pytest Coverage Comment:Begin -->
<!-- Pytest Coverage Comment:End -->
