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
The workflows require a PAT set as secret (see further down for instructions)
See the notes on how to create semantic releases at the bottom of the README.

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
* Fill in your new library values
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

# Protect your main branch
To ensure that only accepted code is put on main, make sure that all changes to main happen using a PR and at least 1
reviewer.
You also want to ensure that no tests are allowed to fail when merging

## Branch Protection
### Ensure branch protection for PRs
In the repo on github go to:
* Settings -> Branches and click "add rule"
* Enable:
  * Require a pull request before merging
    * Require approvals (set the number of required reviewers)
  * Require status checks to pass before merging
    * Require branches to be up to date before merging
  * Require conversation resolution before merging

### Ensure workflow protection
this is not entirely fool proof and secure, but better than nothing, in the repo on github go to:
* Settings -> Actions -> General
* Enable:
  * Allow [owner], and select non-[owner], actions and reusable workflows
* In "Allow specified actions and reusable workflows" add the following string:
  * actions/checkout@v4,
actions/setup-python@v5,
relekang/python-semantic-release@master,
MishaKav/pytest-coverage-comment@main,
actions-js/push@master,
softprops/action-gh-release@v2,

## Create a semantic release PAT and Secrets for the workflow actions
For the semantic release to be able to push new version to the protected branch you need to
create a PAT with the proper permissions and save the pat as a secret in the repo.

### Create PAT
* Click Top right image -> settings
* Developer settings
* Personal access tokens -> Tokens (classic)
* Generate new token -> generate new token (classic)

Settings:
* Note: Semantic release
* Enable:
  * Repo (and all the repo options)
  * workflow
  * admin:repo_hook
* Generate token

Now copy the token (you need this in the next step)

### Create secret
Go to your repo, then:
* Settings -> Secrets -> Actions
* New repository secret
  * Name: SEM_RELEASE
  * Secret: [Your copied PAT token]

The name needs to be the same, as this is what is used in ".github\workflows\semantic-release.yml"


# Semantic release
https://python-semantic-release.readthedocs.io/en/latest/

The workflows are triggered when you merge into main!!

When committing use the following format for your commit message:
* patch:
  `fix: commit message`
* minor:
  `feat: commit message`
* major/breaking (add the breaking change on the third line of the message):
    ```
    feat: commit message

    BREAKING CHANGE: commit message
    ```

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
