# Enterprise DevSecOps CI/CD Pipeline Standard

> Production-Grade CI/CD Pipeline Automation, Security Gates, and Branch Governance.

---

## 🌳 1. Trunk-Based Branching & Governance Policies

To ensure fast iteration, minimal merge conflicts, and zero untested code reaching production, all repositories follow a Trunk-Based Development model with strict branch policies:

```mermaid
gitGraph
    commit id: "v1.2.0"
    branch feature/add-ingress
    checkout feature/add-ingress
    commit id: "feature-code"
    commit id: "unit-tests"
    checkout main
    merge feature/add-ingress id: "PR #42 (Approved & Green)"
    branch release/v1.3.0
    checkout release/v1.3.0
    commit id: "tag v1.3.0"
```

### Branch Policy Protections (`main`)
- **Direct Commits Blocked**: Pull Requests (PR) mandatory for all changes.
- **Reviewer Quorum**: Minimum 2 peer approvals required, including designated **Code Owners**.
- **Build Validation**: PR must pass automated CI pipeline checks before merge is allowed.
- **Short-Lived Feature Branches**: Feature branches live < 48 hours to minimize merge drift.
- **Branch Deletion Guard**: No deletion of `main` / `release/*` branches.

---

## 🔄 2. Modular DevSecOps Pipeline Architecture

Pipelines are orchestrated using reusable YAML templates with built-in security shift-left controls:

```mermaid
flowchart TD
    subgraph PR_Phase["PR Gate (Short Feedback Loop)"]
        A[Git Checkout] --> B[Linting & Formatting<br/>TFLint / Hadolint / ESLint]
        B --> C[Secret Scanning<br/>Gitleaks / Trivy]
        C --> D[SAST & IaC Security<br/>Checkov / tfsec / SonarQube]
        D --> E[Infracost Budget Estimation]
    end

    PR_Phase -->|PR Merged to Main| CD_Phase

    subgraph CD_Phase["Continuous Delivery & Deployment"]
        F[Artifact Build & Container Sign<br/>Docker / Cosign] --> G[Deploy to Dev/UAT<br/>Parallel Automated Test Runs]
        G --> H{Manual Production Gate<br/>Change Advisory Signoff}
        H -->|Approved| I[Blue-Green / Canary Deploy<br/>AKS / Terraform Apply]
        I --> J[Post-Deploy Health Checks]
        J -->|Failed Healthcheck| K[Automated Rollback Hook]
    end
```

---

## 📦 3. Standardized Pipeline Templates

1. **`validate-and-test.yml`**: Validates syntax, runs unit test suites, executes code coverage analysis.
2. **`security-scan.yml`**: Scans container images for CVEs, scans source code for credential leaks, and scans Terraform for CIS benchmark compliance.
3. **`infra-apply.yml`**: Handles state lock acquisition, targeted plans, and controlled apply steps.
4. **`automated-rollback.yml`**: Triggered automatically if synthetic health probes fail post-deployment, restoring previous state or container tags without downtime.

---

## 🔍 4. Automated Drift Detection

- **Daily Scheduled Job**: Runs outside business hours comparing the desired state (Git) against the live Azure environment.
- **Alerting & Remediation**: Automatically raises an incident ticket and notifies DevOps on Slack/Teams if unauthorized drift is detected.
