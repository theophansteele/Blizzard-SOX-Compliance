# Blizzard-SOX-Compliance
#Starter package to help build Compliance as code to support Blizzards SOX efforts 
# Blizzard Entertainment — SOX Compliance as Code

> Terraform-automated SOX ITGC controls across 16 in-scope systems. Replaces manual checkbox evidence collection with continuously-enforced, auditor-queryable artifacts.

[![Terraform](https://img.shields.io/badge/Terraform-≥1.6.0-7c3aed?logo=terraform)](https://www.terraform.io/)
[![Compliance](https://img.shields.io/badge/Framework-SOX%20ITGC-00e5ff)](https://pcaobus.org/Standards/AS/Pages/AS2201.aspx)
[![Controls](https://img.shields.io/badge/Controls-CC6%20·%20CC7%20·%20CC8%20·%20A1%20·%20PI1-10b981)]()
[![Systems](https://img.shields.io/badge/In--Scope%20Systems-16-f59e0b)]()
[![License](https://img.shields.io/badge/Classification-Internal%20Only-ef4444)]()

---

## Table of Contents

- [Overview](#overview)
- [What This Project Does](#what-this-project-does)
- [In-Scope Systems](#in-scope-systems)
- [SOX Controls Coverage](#sox-controls-coverage)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Module Reference](#module-reference)
- [Evidence & Audit Guide](#evidence--audit-guide)
- [GitHub Pages Dashboard](#github-pages-dashboard)
- [Contributing & Change Management](#contributing--change-management)
- [Key Decisions & Design Principles](#key-decisions--design-principles)
- [Contacts](#contacts)

---

## Overview

Blizzard Entertainment is required to comply with the Sarbanes-Oxley Act (SOX) as a subsidiary of Activision Blizzard (NASDAQ: ATVI), now under Microsoft. This repository implements **Compliance as Code** — every SOX IT General Control (ITGC) that can be automated is declared as Terraform, version-controlled, peer-reviewed, and continuously enforced.

**The goal:** eliminate manual evidence collection, reduce audit cycle effort from weeks to hours, and ensure controls are enforced 365 days a year — not just when an auditor asks.

---

## What This Project Does

| Traditional Approach | This Project |
|---|---|
| Screenshots of MFA settings | AWS Config Rule evaluated every 6 hours, immutable result history |
| Spreadsheet of user access list | Terraform state declares every role grant; drift triggers PR |
| 25-ticket sample for change management | 100% of changes require signed commits + 2-reviewer PR approval |
| Annual backup restore test document | AWS Backup job history queryable via CLI; WORM Object Lock retention |
| Quarterly reconciliation spreadsheet | Snowflake task runs daily, results in immutable audit log table |
| Point-in-time encryption screenshots | KMS rotation history cryptographically logged, CLI-queryable |

All 10 control comparisons are documented in the [GitHub Pages dashboard](#github-pages-dashboard).

---

## In-Scope Systems

These 16 systems were identified from Blizzard job descriptions (2023–present) that reference SOX controls and HR/financial systems. All are in scope for this project.

| # | System | Category | Terraform Module |
|---|---|---|---|
| 1 | Amazon Web Services (AWS) | Cloud Platform | `modules/aws` |
| 2 | Google Cloud Platform / Cloud Composer | Cloud Platform | `modules/gcp` |
| 3 | Microsoft Azure | Cloud Platform | `modules/azure` |
| 4 | GitHub | Version Control | `modules/github` |
| 5 | Snowflake | Data Warehouse | `modules/snowflake` |
| 6 | Google BigQuery | Data Warehouse | `modules/bigquery` |
| 7 | Oracle Database | Relational Database | `modules/oracle` |
| 8 | Apache Airflow / Dagster (via Cloud Composer) | Data Orchestration | `modules/gcp` |
| 9 | Docker | Containerization | `modules/aws` |
| 10 | Ansible | Configuration Management | `modules/oracle` |
| 11 | Puppet | Configuration Management | `modules/oracle` |
| 12 | Tableau | BI / Reporting | manual / API |
| 13 | Looker | BI / Reporting | manual / API |
| 14 | Confluence | Collaboration / Docs | `modules/confluence` |
| 15 | Linux (RHEL / Red Hat) | Operating System | `modules/oracle` |
| 16 | Battle.net Platform | Internal Platform | manual / API |

> **Note:** Tableau, Looker, and Battle.net do not have native Terraform providers with sufficient coverage. Controls for these systems are applied via REST API calls (`null_resource` + `local-exec`) or documented as manual procedures with compensating controls.

---

## SOX Controls Coverage

This project targets the following SOC 2 / SOX ITGC control domains:

| Domain | Name | Description |
|---|---|---|
| **CC6** | Logical and Physical Access Controls | IAM, MFA, encryption, access provisioning/deprovisioning |
| **CC7** | System Operations | Monitoring, anomaly detection, incident response |
| **CC8** | Change Management | Authorized, approved, and tested changes |
| **CC9** | Risk Mitigation | Configuration drift detection and remediation |
| **A1** | Availability | Backup, recovery, capacity management |
| **PI1** | Processing Integrity | Data completeness, accuracy, and timeliness |

### Control-to-System Matrix (summary)

| Control | AWS | GCP | Azure | GitHub | Snowflake | BigQuery | Oracle | Linux |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| CC6.1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CC6.2 | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ | ✓ |
| CC6.3 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CC6.6 | ✓ | ✓ | ✓ | | ✓ | ✓ | ✓ | ✓ |
| CC6.7 | ✓ | ✓ | | | ✓ | ✓ | | |
| CC7.1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CC7.2 | ✓ | ✓ | ✓ | | ✓ | ✓ | ✓ | ✓ |
| CC8.1 | ✓ | ✓ | ✓ | ✓ | | | | |
| A1.2 | ✓ | ✓ | ✓ | | | | ✓ | ✓ |
| PI1.1 | | | | | ✓ | ✓ | ✓ | |

Full per-system, per-control implementation detail is in [`controls/sox_controls_mapping.json`](controls/sox_controls_mapping.json) and rendered in the [dashboard](docs/index.html).

---

## Project Structure

```
sox-compliance/
│
├── README.md                          # This file
├── main.tf                            # Root module — calls all SOX modules
├── providers.tf                       # Provider versions + S3 remote state backend
├── variables.tf                       # All input variables with descriptions
├── outputs.tf                         # Key output values (KMS ARNs, bucket names, etc.)
│
├── modules/
│   ├── aws/
│   │   └── main.tf                    # CloudTrail, GuardDuty, Security Hub, Config Rules,
│   │                                  # KMS, S3 WORM, CloudWatch alarms, SNS, Backup, SCPs
│   ├── gcp/
│   │   └── main.tf                    # Cloud KMS, Audit Log sinks, VPC Service Controls,
│   │                                  # Binary Authorization, org policies, log-based alerts
│   ├── azure/
│   │   └── main.tf                    # Key Vault, Defender for Cloud, Log Analytics,
│   │                                  # Activity log alerts, Azure Policy, Backup Vault
│   ├── github/
│   │   └── main.tf                    # Branch protection, signed commits, SSO enforcement,
│   │                                  # secret scanning, sox-engineers + sox-auditors teams
│   ├── snowflake/
│   │   └── main.tf                    # RBAC role hierarchy, network policy, data masking,
│   │                                  # resource monitor, daily reconciliation task
│   ├── bigquery/
│   │   └── main.tf                    # Dataset IAM, CMEK, authorized views, DQ queries
│   ├── oracle/
│   │   └── main.tf                    # Unified Audit policies, TDE, PAM faillock,
│   │                                  # auditd rules, AIDE config, SSH hardening
│   └── confluence/
│       └── main.tf                    # Space permissions, SSO policy via Atlassian API
│
├── controls/
│   └── sox_controls_mapping.json      # Machine-readable controls ↔ systems mapping
│
└── docs/
    └── index.html                     # GitHub Pages dashboard (controls matrix +
                                       # evidence comparison for auditors)
```

---

## Prerequisites

### Required Tooling

| Tool | Minimum Version | Purpose |
|---|---|---|
| [Terraform](https://www.terraform.io/downloads) | `>= 1.6.0` | Infrastructure provisioning |
| [AWS CLI](https://aws.amazon.com/cli/) | `>= 2.0` | AWS authentication + audit queries |
| [gcloud CLI](https://cloud.google.com/sdk/docs/install) | `>= 450.0` | GCP authentication |
| [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) | `>= 2.50` | Azure authentication |
| [GitHub CLI](https://cli.github.com/) | `>= 2.0` | GitHub org management |
| [SnowSQL](https://docs.snowflake.com/en/user-guide/snowsql) | `>= 1.2` | Snowflake administration |

### Required Permissions

You must have the following before running `terraform apply`:

- **AWS:** `AdministratorAccess` on the SOX account (or a scoped policy covering IAM, Config, CloudTrail, KMS, S3, GuardDuty, SecurityHub, Backup, Organizations)
- **GCP:** `roles/owner` on the SOX project + `roles/resourcemanager.organizationAdmin` for org policies
- **Azure:** `Owner` on the SOX subscription + `Global Administrator` in Entra ID for Conditional Access
- **GitHub:** Organization `Owner` role
- **Snowflake:** `ACCOUNTADMIN` role
- **Atlassian:** Organization Admin in Atlassian Access

### Remote State Backend

This project uses S3 + DynamoDB for remote state. Create these before first apply:

```bash
# Create state bucket (versioning + encryption enabled)
aws s3api create-bucket \
  --bucket blizzard-sox-terraform-state \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2

aws s3api put-bucket-versioning \
  --bucket blizzard-sox-terraform-state \
  --versioning-configuration Status=Enabled

aws s3api put-bucket-encryption \
  --bucket blizzard-sox-terraform-state \
  --server-side-encryption-configuration \
  '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'

# Create DynamoDB lock table
aws dynamodb create-table \
  --table-name blizzard-sox-terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-west-2
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/blizzard-entertainment/sox-compliance-iac.git
cd sox-compliance-iac
```

### 2. Create your variables file

Copy the example and fill in your environment-specific values:

```bash
cp terraform.tfvars.example terraform.tfvars
```

```hcl
# terraform.tfvars
environment           = "prod"
aws_account_id        = "123456789012"
gcp_project_id        = "blizzard-sox-prod"
azure_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
github_org            = "blizzard-entertainment"
snowflake_account     = "blizzard.us-west-2"
snowflake_username    = "SOX_TF_ADMIN"

sox_repos = [
  "revenue-data-pipelines",
  "sox-compliance-iac",
  "battle-net-data"
]

sox_admins = [
  "arn:aws:iam::123456789012:user/sox-admin",
  "sox-admin@blizzard.com"
]

alert_email              = "sox-alerts@blizzard.com"
audit_log_retention_days = 2555   # 7 years — SOX 802 requirement
log_archive_bucket       = "blizzard-sox-audit-logs"
```

> **Security:** Never commit `terraform.tfvars` to git. It is in `.gitignore`. Sensitive values (passwords, tokens) should be set as environment variables or pulled from a secrets manager:
> ```bash
> export TF_VAR_snowflake_password=$(aws secretsmanager get-secret-value \
>   --secret-id sox/snowflake-admin-password --query SecretString --output text)
> export TF_VAR_github_token=$(aws secretsmanager get-secret-value \
>   --secret-id sox/github-token --query SecretString --output text)
> ```

### 3. Authenticate to all providers

```bash
# AWS
aws sso login --profile sox-prod

# GCP
gcloud auth application-default login
gcloud config set project blizzard-sox-prod

# Azure
az login
az account set --subscription "blizzard-sox-prod"

# GitHub
gh auth login
```

### 4. Initialize and plan

```bash
terraform init
terraform plan -out=sox-compliance.tfplan
```

Review the plan carefully. All resources are tagged `sox_in_scope = "true"`. Expected on first apply: ~85–110 resources across all modules.

### 5. Apply

```bash
# Apply a single module first to validate (recommended)
terraform apply -target=module.aws_sox

# Full apply
terraform apply sox-compliance.tfplan
```

### 6. Apply Linux/Oracle hardening configs

The `modules/oracle` module generates hardening config files to `modules/oracle/files/`. Deploy these to your hosts via Ansible or Puppet:

```bash
# Via Ansible (recommended)
ansible-playbook -i inventory/sox-hosts.yml \
  playbooks/sox-hardening.yml \
  --extra-vars "auditd_rules=$(terraform output -raw oracle_auditd_rules_path)"
```

---

## Module Reference

### `module.aws_sox`

Implements SOX ITGCs across the AWS account.

| Resource | SOX Control | Purpose |
|---|---|---|
| `aws_kms_key.sox_key` | CC6.6 | Customer-managed encryption key, 90-day auto-rotation |
| `aws_cloudtrail.sox_trail` | CC7.1 | Multi-region trail with log file validation (SHA-256) |
| `aws_guardduty_detector.sox` | CC7.1 | Threat detection across EC2, S3, EKS |
| `aws_securityhub_account.sox` | CC7.1 | Aggregated findings; CIS + NIST benchmarks enabled |
| `aws_config_configuration_recorder.sox` | CC8.1 | Records all resource configuration changes |
| `aws_config_config_rule.*` (7 rules) | CC6.1–CC8.1 | MFA, unused creds, inline policy, SSL, CloudTrail, RDS encryption |
| `aws_cloudwatch_metric_alarm.*` (4 alarms) | CC7.2 | Root usage, IAM changes, KMS deletion, CloudTrail tampering |
| `aws_s3_bucket.audit_logs` | CC6.7/A1.2 | WORM Object Lock, COMPLIANCE mode, 7-year retention |
| `aws_backup_plan.sox` | A1.2 | Daily + monthly backup, lifecycle to cold storage |
| `aws_organizations_policy.deny_disable_cloudtrail` | CC7.1 | SCP prevents disabling CloudTrail/Config in any account |

**Key outputs:** `sox_kms_key_arn`, `sox_audit_bucket_name`, `cloudtrail_arn`, `sox_sns_topic_arn`

---

### `module.gcp_sox`

Implements SOX ITGCs across the GCP project.

| Resource | SOX Control | Purpose |
|---|---|---|
| `google_kms_crypto_key.sox` | CC6.6 | CMEK key, 90-day rotation, `prevent_destroy = true` |
| `google_project_iam_audit_config.*` (4) | CC7.1 | DATA_READ + DATA_WRITE audit logs for BQ, GCS, IAM, Composer |
| `google_logging_project_sink.sox_audit_sink` | CC7.1 | All audit logs → locked GCS bucket (7-year retention policy) |
| `google_access_context_manager_service_perimeter.sox_perimeter` | CC6.6 | VPC Service Controls around BigQuery, GCS, Composer |
| `google_organization_policy.disable_sa_key_creation` | CC6.1 | Prevents service account key creation org-wide |
| `google_organization_policy.require_binary_auth` | CC8.1 | All container workloads require Binary Authorization |
| `google_monitoring_alert_policy.sox_iam_changes` | CC7.2 | Alert on any IAM policy or service account change |

**Key outputs:** `sox_kms_key_id`, `sox_audit_bucket_name`, `sox_log_sink_name`

---

### `module.azure_sox`

Implements SOX ITGCs across the Azure subscription.

| Resource | SOX Control | Purpose |
|---|---|---|
| `azurerm_key_vault.sox` | CC6.6 | Premium Key Vault, purge protection on, network deny-by-default |
| `azurerm_security_center_subscription_pricing.*` (4) | CC7.1 | Defender for Servers, Storage, SQL, Key Vaults |
| `azurerm_log_analytics_workspace.sox` | CC7.1 | Centralized log workspace (730 days online + archive) |
| `azurerm_monitor_activity_log_alert.*` (3) | CC7.2 | Alerts: policy changes, RBAC changes, Key Vault deletion |
| `azurerm_policy_assignment.*` (2) | CC8.1 | Enforce sox_in_scope tag + HTTPS-only storage |
| `azurerm_management_lock.sox_rg_lock` | CC8.1 | CanNotDelete lock on SOX resource group |
| `azurerm_data_protection_backup_vault.sox` | A1.2 | GeoRedundant backup vault |
| `azurerm_storage_account.sox_archive` | CC7.1 | GRS archive for logs beyond 730-day LAW limit |

**Key outputs:** `sox_keyvault_uri`, `sox_log_workspace_id`, `sox_storage_archive_id`

---

### `module.github_sox`

Enforces SOX change management controls across all SOX-scoped repositories.

| Resource | SOX Control | Purpose |
|---|---|---|
| `github_organization_settings.sox` | CC6.1/CC6.2 | Default repo permission: none; SSO required; members cannot create public repos |
| `github_branch_protection.sox_main` | CC8.1 | 2 reviewers, CODEOWNERS, signed commits, no force push on `main` |
| `github_branch_protection.sox_release` | CC8.1 | 2 reviewers, signed commits on `release/*` branches |
| `github_repository_security_and_analysis.sox` | CC7.1 | Advanced Security, secret scanning, push protection on all SOX repos |
| `github_team.sox_engineers` | CC6.3 | Scoped team with `push` access to SOX repos |
| `github_team.sox_auditors` | CC6.1 | Read-only team for internal/external auditors |

**Key outputs:** `sox_engineers_team_id`, `sox_auditors_team_id`, `codeowners_template`

---

### `module.snowflake_sox`

Enforces access, encryption, monitoring, and processing integrity controls in Snowflake.

| Resource | SOX Control | Purpose |
|---|---|---|
| `snowflake_role.sox_*` (4 roles) | CC6.1 | SOX_ADMIN, SOX_DATA_ENGINEER, SOX_ANALYST, SOX_AUDITOR |
| `snowflake_network_policy.sox` | CC6.2 | Restrict connections to Blizzard CIDR ranges only |
| `snowflake_masking_policy.pii_email` | CC6.6 | Mask email addresses from non-admin roles |
| `snowflake_masking_policy.financial_amount` | CC6.6 | Mask revenue amounts from analyst roles |
| `snowflake_resource_monitor.sox` | CC7.2 | Alert at 75%/90% credit usage; suspend at 100% |
| `snowflake_task.daily_revenue_reconciliation` | PI1.1 | Daily 06:00 UTC reconciliation; results in AUDIT_LOG schema |
| `snowflake_warehouse.sox_audit` | CC7.1 | Dedicated warehouse for auditor queries |

**Key outputs:** `sox_database_name`, `sox_admin_role_name`, `sox_auditor_role_name`, `sox_network_policy`

---

### `module.bigquery_sox`

Enforces data access, encryption, and processing integrity controls in BigQuery.

| Resource | SOX Control | Purpose |
|---|---|---|
| `google_bigquery_dataset.sox_revenue` | CC6.1 | SOX-scoped dataset with CMEK and restricted IAM |
| `google_bigquery_table.sox_authorized_view` | CC6.7 | Masks revenue_amount from non-SOX-admin callers |
| `google_bigquery_data_transfer_config.sox_dq_check` | PI1.1 | Daily DQ check inserts null/negative count to `dq_results` |

---

### `module.oracle_sox` (Oracle + Linux + Ansible + Puppet)

Generates configuration files for Oracle Database and Linux hosts. Files are deployed via Ansible/Puppet.

| Generated File | SOX Control | Deploy To |
|---|---|---|
| `files/sox-auditd.rules` | CC7.1 | `/etc/audit/rules.d/sox.rules` on all Linux hosts |
| `files/sox-pam-faillock.conf` | CC6.3 | `/etc/security/faillock.conf` on all Linux hosts |
| `files/sox-sshd_config` | CC6.1/CC6.6 | `/etc/ssh/sshd_config.d/sox.conf` on all Linux hosts |
| `files/sox-aide.conf` | CC7.2 | `/etc/aide.conf` on all Linux hosts |

Oracle Unified Audit policies are applied via `null_resource` + `sqlplus`. See module for SQL statements.

**Key outputs:** `auditd_rules_path`, `aide_config_path`, `sshd_config_path`, `pam_config_path`

---

### `module.confluence_sox`

Applies SOX space permissions and SSO enforcement via the Atlassian REST API.

| Action | SOX Control | Method |
|---|---|---|
| Restrict SOX space to `sox-auditors` group | CC6.1 | Atlassian Confluence REST API (PUT /space/SOX/permission) |
| Remove anonymous access from SOX space | CC6.1 | Atlassian Confluence REST API (DELETE) |
| Enforce SAML SSO for all org members | CC6.2 | Atlassian Admin API (POST /admin/v1/orgs/:id/policies) |
| SOX control documentation page template | CC8.1 | Generated as `files/sox-page-template.txt` |

---

## Evidence & Audit Guide

When auditors request evidence, point them to these sources directly. No screenshots needed.

### Access Controls (CC6.1 / CC6.3)

```bash
# AWS — who has access to what
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d

# Snowflake — current role grants (run as SOX_AUDITOR role)
SHOW GRANTS ON DATABASE REVENUE_REPORTING;
SHOW GRANTS TO ROLE SOX_ANALYST;

# GitHub — team membership
gh api /orgs/blizzard-entertainment/teams/sox-data-engineers/members
```

### MFA Enforcement (CC6.2)

```bash
# AWS Config — compliance status for all IAM users
aws configservice get-compliance-details-by-config-rule \
  --config-rule-name sox-mfa-enabled-for-iam-console-access \
  --compliance-types NON_COMPLIANT

# Returns: list of non-compliant users — empty list = full compliance
```

### Encryption (CC6.6)

```bash
# AWS KMS — key rotation history
aws kms list-key-rotations --key-id alias/sox-compliance-key

# GCP — CMEK key metadata
gcloud kms keys describe sox-data-key \
  --keyring sox-compliance-keyring \
  --location us-central1

# Azure — Key Vault key rotation policy
az keyvault key rotation-policy show \
  --vault-name kv-sox-prod --name sox-data-key
```

### Audit Logs (CC7.1)

```bash
# AWS CloudTrail — query all console logins in a period
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin \
  --start-time 2025-01-01T00:00:00Z \
  --end-time 2025-03-31T23:59:59Z

# Validate log file integrity (tamper detection)
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:us-west-2:123456789012:trail/sox-compliance-trail \
  --start-time 2025-01-01T00:00:00Z
```

### Change Management (CC8.1)

```bash
# GitHub — all merged PRs to main for a SOX repo with approvers
gh pr list --repo blizzard-entertainment/revenue-data-pipelines \
  --state merged --base main \
  --json number,title,mergedAt,reviews \
  --jq '.[] | {pr: .number, title: .title, merged: .mergedAt, approvers: [.reviews[].author.login]}'

# Verify signed commits
git log --show-signature --since="2025-01-01" origin/main
```

### Backups (A1.2)

```bash
# AWS Backup — job history for SOX vault
aws backup list-backup-jobs \
  --by-backup-vault-name sox-backup-vault \
  --by-state COMPLETED \
  --by-created-after 2025-01-01T00:00:00Z \
  --query 'BackupJobs[*].{Resource:ResourceArn,Status:State,Completed:CompletionDate}'
```

### Processing Integrity (PI1.1)

```sql
-- Snowflake — query reconciliation results directly (run as SOX_AUDITOR)
SELECT
  check_time,
  record_count,
  null_amounts,
  negative_amounts,
  total_revenue
FROM REVENUE_REPORTING.AUDIT_LOG.DATA_QUALITY_CHECKS
WHERE check_time >= '2025-01-01'
ORDER BY check_time DESC;

-- Flag any days with data quality failures
SELECT check_time, null_amounts, negative_amounts
FROM REVENUE_REPORTING.AUDIT_LOG.DATA_QUALITY_CHECKS
WHERE null_amounts > 0 OR negative_amounts > 0;
```

---

## GitHub Pages Dashboard

The `docs/index.html` file is a self-contained dashboard deployed via GitHub Pages. It provides:

- **SOX Controls Matrix** — all 16 systems, their mapped controls, and per-control implementation detail. Click any row to expand.
- **Terraform Project Structure** — file-by-file breakdown of what each module manages.
- **Evidence Comparison** — side-by-side view of old manual checkbox evidence vs. new automated artifact for each control. Filterable by domain. Designed for auditors to understand the approach in a single page.

### Deploy to GitHub Pages

1. Push this repository to GitHub
2. Go to **Settings → Pages**
3. Set Source: **Deploy from a branch**, Branch: `main`, Folder: `/docs`
4. The dashboard will be live at `https://blizzard-entertainment.github.io/sox-compliance-iac/`

> Access can be restricted to internal users via GitHub Enterprise's IP allowlist or SSO requirement — recommended for a SOX-classified repository.

---

## Contributing & Change Management

All changes to this repository are themselves SOX change management evidence. The following rules are enforced at the GitHub API level and **cannot be bypassed**:

1. **No direct commits to `main` or `release/*`** — all changes via pull request
2. **2 approvals required** — at least one must be a CODEOWNER (`sox-admins` group)
3. **Signed commits required** — configure GPG signing: `git config --global commit.gpgsign true`
4. **All status checks must pass** — `sox-compliance-check`, `security-scan`, `unit-tests`
5. **Stale reviews dismissed** — new commits reset approval count

### Development Workflow

```bash
# 1. Create a feature branch
git checkout -b feature/sox-CC6-1-snowflake-rbac-update

# 2. Make changes
# Edit modules/snowflake/main.tf

# 3. Run terraform plan and include output in PR description
terraform plan -target=module.snowflake_sox > plan-output.txt

# 4. Commit with sign-off
git commit -S -m "feat(snowflake): add SOX_AUDITOR role for external audit team

SOX Control: CC6.1 — Least-privilege read-only role for Q1 2026 external audit.
Ticket: SOX-2026-Q1-001
"

# 5. Open PR — plan output must be included in description
gh pr create --title "feat(snowflake): SOX_AUDITOR role for Q1 2026 audit" \
  --body "$(cat pr-template.md)"
```

### PR Template

All PRs to this repository should include:

```markdown
## SOX Change Record

**Control(s) affected:** CC6.1
**System(s) affected:** Snowflake
**Change type:** [ ] New control  [x] Modify existing  [ ] Remediation
**Ticket/JIRA:** SOX-2026-Q1-001
**Risk assessment:** Low — adding read-only role, no changes to existing permissions

## Terraform Plan Output
<details>
<summary>terraform plan</summary>

```
(paste plan output here)
```
</details>

## Testing
- [ ] `terraform plan` reviewed and attached
- [ ] Change tested in staging environment
- [ ] Rollback procedure documented
- [ ] SOX control owner notified
```

---

## Key Decisions & Design Principles

**1. Immutability over configuration**
Audit evidence must be tamper-evident. All controls use immutable artifacts: WORM S3 Object Lock in COMPLIANCE mode, CloudTrail SHA-256 log file validation, signed Git commits. These cannot be altered even by administrators.

**2. 7-year retention everywhere**
SOX Section 802 requires records be retained for 7 years. All log sinks, backup vaults, and storage buckets are configured for 2,555 days minimum. This is enforced at the storage layer, not just by policy.

**3. Least privilege by default**
GitHub organization default permission is `none`. Snowflake financial amounts are masked at the database layer. AWS IAM uses deny-by-default with explicit grants. Access must be declared in code and peer-reviewed.

**4. Auditor self-service**
Auditors are provisioned with read-only access (Snowflake `SOX_AUDITOR` role, GitHub `sox-auditors` team, AWS read-only policy). All evidence queries in this README can be run directly by auditors without involving the engineering team, eliminating the evidence collection bottleneck.

**5. Controls as code = controls as proof**
The Terraform state file and git history are themselves audit evidence. Every control declaration was peer-reviewed and approved before being applied. Drift from declared state triggers automated alerts.

---

## Contacts

| Role | Team | Contact |
|---|---|---|
| SOX Control Owner | InfoSec / Compliance | sox-control-owner@blizzard.com |
| Terraform IaC Owner | Platform Engineering | platform-eng@blizzard.com |
| SOX Audit Liaison | Finance / Internal Audit | sox-audit@blizzard.com |
| Security Incidents | InfoSec | security@blizzard.com |
| SOX Alerts (automated) | PagerDuty + Email | sox-alerts@blizzard.com |

---

*Classification: Internal Only — Do not distribute outside Blizzard Entertainment.*
*Last updated: June 2026 · Framework: SOX ITGC / SOC 2 Type II · Terraform ≥ 1.6.0*
