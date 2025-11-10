# Security Automation and Threat Modelling Program

This repository houses the organisation-wide security automation assets that every project can inherit. The guidance below explains the reusable workflows available in [`/.github/workflows`](./workflows) as well as expectations for threat modelling and security reporting across your codebases.

## Reporting a Vulnerability

If you discover a vulnerability in any repository under this organisation, please 
report it privately to Jordan. Provide enough detail to help reproduce the issue (steps to reproduce, affected versions, proof-of-concept).
Receipt acknowledgment will be within **2 business days** and an aim to provide a fix or mitigation wll be within **14 days** for critical issues.

## Required Automated Checks

Every repository SHOULD include the following workflows by referencing them via the
[`uses` keyword](https://docs.github.com/actions/using-workflows/reusing-workflows):

| Concern | Workflow | Description |
| --- | --- | --- |
| Static Application Security Testing | `prose-intelligence-ltd/.github/.github/workflows/codeql.yml@main` | Runs CodeQL with configurable languages and build mode. |
| Dependency & License Review | `prose-intelligence-ltd/.github/.github/workflows/dependency-review.yml@main` | Enforces dependency vulnerability and licence checks on pull requests. |
| Secret Scanning | `prose-intelligence-ltd/.github/.github/workflows/gitleaks.yml@main` | Executes Gitleaks to prevent committed credentials. |
| Infrastructure as Code Scanning | `prose-intelligence-ltd/.github/.github/workflows/checkov.yml@main` | Scans Terraform/Kubernetes/CloudFormation templates with Checkov. |
| Threat Model Validation | `prose-intelligence-ltd/.github/.github/workflows/threat-model-lint.yml@main` | Ensures the threat model document exists and contains required sections. |

Example workflow wiring multiple checks together:

```yaml
name: security-suite
on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # nightly baseline

jobs:
  sast:
    uses: org/.github/.github/workflows/codeql.yml@main
    with:
      languages: '["python"]'

  dependency-review:
    if: github.event_name == 'pull_request'
    uses: org/.github/.github/workflows/dependency-review.yml@main

  gitleaks:
    uses: org/.github/.github/workflows/gitleaks.yml@main

  checkov:
    uses: org/.github/.github/workflows/checkov.yml@main
    with:
      framework: terraform,kubernetes
      directory: infrastructure

  threat-model-lint:
    uses: org/.github/.github/workflows/threat-model-lint.yml@main
    with:
      path: docs/threat-model.md
```

## Threat Modelling Expectations

Each repository MUST maintain an up-to-date threat model stored alongside the code (preferably under `docs/threat-model.md`). At a minimum the document should contain:

1. **System Overview** – architecture description, diagrams, and trust boundaries.
2. **Assets** – critical data, secrets, and services that require protection.
3. **Threats** – identified issues mapped to a framework such as STRIDE.
4. **Mitigations** – implemented or planned countermeasures.
5. **Decisions** – accepted risks, open questions, and follow-up tasks.

The `threat-model-lint` workflow leverages [`scripts/validate-threat-model.sh`](./scripts/validate-threat-model.sh)
which validates the presence of these sections and ensures the file exists. Repositories can extend the script to include custom linting.
