# 🔐 DevSecOps Automation Pipeline

## 📌 Overview

The **DevSecOps Automation Pipeline** is a GitLab CI/CD pipeline built to automate security scanning, linting, and static code analysis for Terraform and Ansible projects.
It ensures that infrastructure-as-code (IaC) configurations comply with security best practices, reducing vulnerabilities and operational risks before deployment.

---

## 🛠️ Tools & Technologies Used

| Tool         | Description                                                                 |
|--------------|-----------------------------------------------------------------------------|
| Terraform    | Infrastructure-as-Code (IaC)                                                |
| Ansible      | Automation and configuration management                                     |
| tflint       | Linting and best practices for Terraform                                    |
| ansible-lint | Detects problems in Ansible playbooks                                       |
| Trivy        | Vulnerability scanner for container images, filesystems, and IaC            |
| Checkov      | Infrastructure as Code (IaC) static analysis to find misconfigurations      |
| Semgrep      | Lightweight static analysis for code security (SAST)                        |
| Gitleaks     | Detects hardcoded secrets in source code                                    |
| Pre-commit   | Ensures code quality and security before commits                            |
| Docker       | Used to test and run tools in isolation without system installation         |
| GitLab CI/CD | Automates security checks on every commit/push                              |

---

## 📂 Project Structure

├── bootstrap │
├   ── bootstrap.sh
│   └── README.md
├── Dockerfile
├── generate_trivy_report.sh
├── README.md
├── sast_results.json
└── trivy_ansible.json

markdown
Copy
Edit

---

## 🔧 Setup & First Steps

1. **Create the Repository**
   - Create a GitLab repository named `DevSecOps` under your group.
   - Initialize it with `.gitignore` for Terraform/Ansible and a basic README.

2. **Add Pre-Commit Config**
   - Create `.pre-commit-config.yaml` to include:
     - tflint
     - ansible-lint
     - gitleaks
     - checkov
     - semgrep

3. **Write CI/CD Config**
   - Add `.gitlab-ci.yml` with pipeline stages:
     - Lint
     - SAST
     - Vulnerability Scan
     - Report

4. **Create Docker Image**
   - Build Dockerfile with all required tools pre-installed
   - Push it to GitLab container registry for reuse in CI and local testing

5. **Test Locally**
   - Validate that the pipeline runs locally using the Docker image

---

## 🧪 How to Use This Project

### 1️⃣ Clone the Repository

    git clone https://gitlab.snapp.ir/sysops/ansible-automation/DevSecOps
    cd DevSecOps
### 2️⃣ Run Locally Using Docker (No Local Tool Installation)
    docker pull registry.snapp.tech/sysops/ansible-automation/devsecops:1.0
    docker run --rm -v $(pwd):/project -w /project \
    registry.snapp.tech/sysops/ansible-automation/devsecops:1.0 \
    bash -c "pre-commit run --all-files && ./generate_trivy_report.sh"
### 3️⃣ Pre-Commit Checks
    pre-commit run --all-files
    This step ensures security and code quality checks before each commit.

# 🔒 Security Tools Details
## ✅ Trivy
#### Scans Terraform and Ansible files for vulnerabilities and misconfigurations.

## ✅ Checkov
#### Enforces security policies and best practices for IaC.

## ✅ Semgrep
#### Static application security testing (SAST) for detecting insecure code patterns.

## ✅ Gitleaks
#### Detects secrets in Git history or codebase (tokens, keys, passwords).

# Run manually with Docker:
    docker run --rm -v $(pwd):/code zricethezav/gitleaks detect --source=/code
**Add to pre-commit:**
    - repo: https://github.com/gitleaks/gitleaks
      rev: v8.18.1
      hooks:
        - id: gitleaks
# 📄 Output Reports

## Tool	Report File
### Checkov	checkov_results.json
### Trivy	trivy_ansible.json
### Semgrep	sast_results.json
### Gitleaks	gitleaks_report.txt
### TFLint	tflint_report.txt (opt)

# 🔁 GitLab CI/CD Triggering From Another Repo
    To trigger this pipeline from another project and make it required before continuing, do the following:

## Step 1: Use trigger in .gitlab-ci.yml of the second repo
    trigger-devsecops:
      stage: security
      trigger:
        project: sysops/ansible-automation/DevSecOps
       branch: main
     allow_failure: false

## Step 2: Grant Access
    Ensure both projects are in the same GitLab group
    Enable group runners
    Make sure the DevSecOps repo is visible to the triggering project or grant access tokens


# 🔐 Pre-commit Hook Setup for DevSecOps

This project uses pre-commit to automatically run security and linting checks before you commit code. This helps ensure that issues are caught early, and code quality is maintained.

### ✅ Why Use Pre-commit?
- Catches issues before they reach CI/CD pipelines
- Saves time by checking only the changes you make
- Maintains code consistency and security

---

### 🧩 Requirements

Make sure you have the following installed:

- Python 3.7+
- pip
- git
- pre-commit

---

### ⚙️ Installation

Install pre-commit globally:

```bash
pip install pre-commit
```

Or install it only for your project:

```bash
python3 -m venv venv
source venv/bin/activate
pip install pre-commit
```

---

### 🔁 Setup in Your Project

1. **Add a `.pre-commit-config.yaml` file** to your project root.

   Example `.pre-commit-config.yaml`:

   ```yaml
   ---
   repos:
     # Terraform formatting & linting
     - repo: https://github.com/antonbabenko/pre-commit-terraform
       rev: v1.79.0
       hooks:
         - id: terraform_fmt
         - id: terraform_validate
         - id: terraform_tflint

     # Ansible lint
     - repo: https://github.com/ansible-community/ansible-lint
       rev: v6.21.1
       hooks:
         - id: ansible-lint
           args: ['--nocolor', '-q']
           files: ^ansible/|\.ya?ml$

     # Trivy wrapper for IaC scanning
     - repo: local
       hooks:
         - id: trivy-config
           name: Trivy Config Scan
           entry: bash -c 'trivy config --quiet .'
           language: system
           pass_filenames: false
           files: \.(tf|ya?ml)$

     # Semgrep SAST
     - repo: https://github.com/returntocorp/semgrep
       rev: v1.60.0
       hooks:
         - id: semgrep
           args: ['--config', 'auto', '--quiet']
           exclude: ^tests/

     # Gitleaks secrets scan
     - repo: https://github.com/gitleaks/gitleaks
       rev: v8.18.1
       hooks:
         - id: gitleaks
           args: ['protect', '--redact']
           pass_filenames: false

     # Checkov security for Terraform/Ansible
     - repo: https://github.com/bridgecrewio/checkov
       rev: 3.1.42
       hooks:
         - id: checkov
           args: ['--quiet', '--output', 'json']
           pass_filenames: false

     # General YAML & Git hygiene
     - repo: https://github.com/adrienverge/yamllint
       rev: v1.32.0
       hooks:
         - id: yamllint
           args: ['-d', '{extends: default, rules: {line-length: disable}}']
     - repo: https://github.com/pre-commit/pre-commit-hooks
       rev: v4.5.0
       hooks:
         - id: end-of-file-fixer
         - id: trailing-whitespace
         - id: check-yaml
         - id: check-merge-conflict
   ```

2. **Install Git hook scripts:**

   Run the following command to initialize pre-commit and install the Git hooks:

   ```bash
   pre-commit install
   ```

   This installs the necessary Git hooks so that checks run before every commit.

---

### 🚀 Run It Manually

To run all pre-commit hooks manually on all files:

```bash
pre-commit run --all-files
```

To run pre-commit hooks only on staged files (i.e., files that are ready to be committed):

```bash
pre-commit run
```

### 📋 Redirect Output to a File

If you want to save the output of your pre-commit run to a file (e.g., `pre-commit-result.txt`), run:

```bash
pre-commit run --all-files | tee pre-commit-result.txt
```

---

### 💡 Best Practices

- **Always pin hook versions** using `rev`. This ensures that you get the same version of hooks each time.
- **Keep hooks specific and minimal** — only include hooks that are needed to avoid slowing down the process.
- Use **quiet/JSON formats** for faster parsing in CI/CD to speed up feedback.
- Add `pre-commit run --all-files` to your CI/CD pipeline to enforce security checks before merging code.

---

### 🔗 More Resources

- [Pre-commit Docs](https://pre-commit.com/)
- [Ansible Lint](https://github.com/ansible-community/ansible-lint)
- [TFLint](https://github.com/terraform-linters/tflint)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Semgrep](https://semgrep.dev/)
- [Checkov](https://github.com/bridgecrewio/checkov)
- [Yamllint](https://github.com/adrienverge/yamllint)

---
# 🔐 Secrets and Security Scanning Ignore Configuration

This project uses [Gitleaks](https://github.com/gitleaks/gitleaks) and [Trivy](https://github.com/aquasecurity/trivy) for secrets and security scanning. To customize and control false positives or acceptable findings, you can use the following ignore files:

---

## 📁 .gitleaksignore

**Purpose**: To suppress known false positives or acceptable secrets during Gitleaks scans.

### 🧪 Sample `.gitleaksignore`

```json
{
  "allowlist": {
    "description": "Allowed secrets for development purposes",
    "regexes": [
      "example_key_[0-9]+",
      "dummy_secret"
    ],
    "paths": [
      "tests/",
      "scripts/example.sh"
    ]
  }
}
```

### ✅ How to Use

1. Create a `.gitleaksignore` file in the root of your project.
2. Add regex patterns for secrets you want to allow.
3. Optionally, specify paths where these false positives are allowed.

### 🏃 Run Gitleaks with Ignore

```bash
gitleaks detect --source . --report-format=json --report-path=gitleaks_report.json --config-path=.gitleaksignore
```

---

## 🛡️ .trivyignore

**Purpose**: To exclude specific vulnerabilities or misconfigurations in Trivy scans that are considered false positives or acceptable risks.

### 🧪 Sample `.trivyignore`

```txt
AVD-DS-0002 # USER instruction missing in Dockerfile (acceptable for this base image)
AVD-DS-0026 # Missing HEALTHCHECK instruction
```

### ✅ How to Use

1. Create a `.trivyignore` file in your project root.
2. Add each vulnerability/misconfiguration ID on a new line.
3. Optionally, add a comment for clarity.

### 🏃 Run Trivy with Ignore

```bash
trivy config . --ignorefile .trivyignore --format table
```

Or in Docker image scanning:

```bash
trivy image --ignorefile .trivyignore my-docker-image:latest
```

---

## 📌 Notes

- Use these ignore files **only** after manual verification of the findings to avoid ignoring real issues.
- Document reasons in comments to maintain transparency for security reviews.
- Keep the ignore files under version control to track changes over time.

# DevSecOps Pipeline - Renovate, Grype & SBOM

This document outlines the setup and usage of the **Renovate**, **Grype vulnerability scanner**, and **SBOM (Software Bill of Materials)** scans integrated into the CI/CD pipeline.

---

## 1. Renovate Setup

### Purpose
Automatically keeps dependencies and Docker images up-to-date with merge requests created by Renovate Bot.

### Configuration
- Renovate config is stored in `renovate.json` at the root of the repo.
- Automerges safe updates, groups updates by package types, and respects schedules.

### Running Renovate
- Renovate runs in a **separate scheduled pipeline** (`.gitlab-ci-renovate.yml`) triggered weekly.
- The pipeline creates merge requests for dependency updates.
- MR comments and auto-merge logic are handled by shell scripts in `merge/auto_merge.sh` and `merge/comment_on_mr.sh`.

---

## 2. Grype Vulnerability Scanning

### Purpose
Detects vulnerabilities in container images, dependencies, and the codebase using the Grype scanner.

### Configuration
- Grype is invoked in the CI pipeline in the `sbom_grype_scan` job.
- It uses SBOM data generated by the `sbom_scan.sh` script.
- The scan results are stored in JSON or summary text files (`grype_report.json`).

### Usage
- Run automatically on merge requests and main branch.
- Reports are stored as pipeline artifacts for review.
- Vulnerabilities can be posted as comments on merge requests for visibility.

---

## 3. SBOM (Software Bill of Materials)

### Purpose
Generate a detailed inventory of all software components and dependencies used in the project.

### Configuration
- The SBOM generation script (`sbom_scan.sh`) collects component metadata.
- The generated SBOM file is used as input for Grype vulnerability scanning.
- SBOM reports are saved as artifacts in the pipeline.

---

## 4. Pipeline Details

### Main Pipeline (`.gitlab-ci.yml`)
- Runs linting and security scans: Terraform, Ansible, Semgrep, Trivy, Gitleaks.
- Does NOT include Renovate pipeline to keep independence.

### Renovate Pipeline (`.gitlab-ci-renovate.yml`)
- Runs on a scheduled pipeline only (configured in GitLab schedules).
- Handles dependency update merge requests and auto-merging.

### Merge Request Commenting
- Automated MR comments for Renovate updates, Grype, and SBOM scan results.
- Uses GitLab API with `GITLAB_TOKEN`.

---

## 5. How to Maintain

- **Update `renovate.json`** to customize Renovate behavior, grouping, schedules, and auto-merge rules.
- **Keep `sbom_scan.sh` and Grype version up to date** for accurate vulnerability detection.
- **Check pipeline schedules regularly** and adjust timing as needed.
- **Monitor Renovate MRs and vulnerability reports** in merge requests.

---

## 6. Credentials and Access

- Make sure `GITLAB_TOKEN` is set as a protected variable in CI/CD settings with API scope.
- If using private Docker registries, configure access credentials properly for scanning tools.

---

## 7. References

- [Renovate Documentation](https://docs.renovatebot.com/)
- [Grype Vulnerability Scanner](https://github.com/anchore/grype)
- [GitLab Pipeline Schedules](https://docs.gitlab.com/ee/ci/pipelines/schedules.html)
- [SBOM Standards and Tools](https://cyclonedx.org/)

---

## 8. Contact

For help or improvements, contact @me please = soroush.man.hd@gmail.com .
