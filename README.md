# Cloudflare DNS Records Management System

Automated DNS configuration synchronization between YAML files and Cloudflare domains using Git-driven workflows and Ansible automation.

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Requirements](#requirements)
- [Repository Structure](#repository-structure)
- [Branch Workflow](#branch-workflow)
- [Automation Pipeline](#automation-pipeline)
- [Security Implementation](#security-implementation)
- [Best Practices](#best-practices)
- [Troubleshooting Guide](#troubleshooting-guide)

## Overview

This repository implements a declarative DNS management system for Cloudflare domains through:

1. YAML-based configuration files
2. Git version control for audit trails
3. Automated DNS (A,AAA,CNAME) synchronization via Ansible/Semaphore
4. Orphan branch per-domain isolation strategy

## Key Features

- **Git-controlled DNS**: Full version history of DNS changes
- **Declarative management**: Desired state defined in `records.yaml`
- **Automated sync**: Real-time updates through webhook triggers
- **Domain isolation**: Separate branches prevent cross-domain conflicts
- **Audit-ready**: Complete change history through Git commits

## Requirements

**Local Environment Setup**:

1. [SCM](https://about.gitea.com/) - Repository hosting with webhook capabilities
2. [Yamllint](https://yamllint.readthedocs.io/) - Configuration validation

**External Services**:

1. [Ansible Semaphore](https://semaphoreui.com/) - Automation orchestration
2. [Ansible Cloudflare Collection](https://docs.ansible.com/ansible/latest/collections/community/general/cloudflare_dns_module.html) - Ansible Collection
3. [Cloudflare Account & API Token](https://developers.cloudflare.com/api/) - API integration

## Repository Structure

Each domain exists in its own orphan branch containing only one file:

```mermaid
graph TD
    A[Main Branch] --> B(orphan branches)
    B --> C[example.com]
    C --> D[records.yaml]
    B --> E[test.org]
    E --> F[records.yaml]
```

**records.yaml** format:

```yaml
# Cloudflare DNS configuration for the zone
cloudflare_zone: example.net
cloudflare_zone_id: YOUR_ZONE_ID_HERE  # Found on the Cloudflare Dashboard → Overview page

records:
  # A record (IPv4 address)
  - type: A
    name: www                 # Full domain: www.example.net
    content: 192.0.2.1          # IPv4 address
    proxied: true               # true = proxied through Cloudflare, false = DNS only

  # CNAME record (alias)
  - type: CNAME
    name: blog                # Full domain: blog.example.net
    content: example.net        # Canonical name to point to
    proxied: false              # CNAMEs can also be proxied or DNS only

  # AAAA record (IPv6 address)
  - type: AAAA
    name: ipv6                # Full domain: ipv6.example.net
    content: 2001:db8::1        # IPv6 address
    proxied: true
```

## Branch Workflow

Git branch strategy for domain management:

```mermaid
sequenceDiagram
    participant User
    participant Git
    participant Semaphore
    participant Ansible
    participant Cloudflare

    User->>Git: Create orphan branch
    Note left of Git: git checkout --orphan domain.com

    Git->>User: Initialize branch
    User->>Git: Add records.yaml
    User->>Git: Commit configuration changes
    User->>Git: Push to origin

    Git->>Semaphore: Trigger webhook
    Note left of Semaphore: Webhook URL configured in repo settings

    Semaphore->>Ansible: Start DNS sync playbook
    Ansible->>Cloudflare: Authenticate API
    Cloudflare->>Ansible: Validate credentials
    Ansible->>Cloudflare: Update DNS records
    Cloudflare->>Ansible: Return sync status
    Ansible->>Semaphore: Report completion
    Semaphore->>User: Notify success/failure
```

**Branch Creation Process**:

```bash
# Create new domain branch
git checkout --orphan example.com
# Remove existing files
git rm -rf .
# Create records.yaml
echo "records: []" > records.yaml
# Commit initial structure
git add records.yaml && git commit -m "Initial commit"
# Push to origin
git push origin example.com
```

## Automation Pipeline

Three-stage synchronization process:

1. **Configuration Stage**:

   ```mermaid
   graph LR
       A[Edit records.yaml] --> B[Validate with yamllint]
       B --> C[Commit changes]
   ```

2. **Trigger Stage**:

   ```mermaid
   graph LR
     A[Git Commit] --> B(Semaphore Webhook)
     B --> C[Ansible Playbook Execution]
   ```

3. **Execution Stage**:
   ```mermaid
   graph LR
     C[Ansible Playbook] --> D[Cloudflare API Auth]
     D --> E[DNS Record Sync]
     E --> F[Audit Log Creation]
   ```

## Security Implementation

Critical security measures:

- **Credential Storage**: Cloudflare API tokens stored in Semaphore's encrypted vault
- **Branch Protection**: Require pull requests for production branch changes
- **Audit Trail**: Git commit history tracks all configuration modifications
- **Secret Rotation**: Regular token rotation through Semaphore UI

**Credential Setup**:

1. Generate Cloudflare API token with DNS permissions
2. In Semaphore UI:
   - Navigate to Project Settings > Credentials
   - Add new credential with Cloudflare token
   - Link to Ansible playbook environment variables

## Best Practices

| Practice              | Implementation                     | Rationale                       |
| --------------------- | ---------------------------------- | ------------------------------- |
| One Domain Per Branch | Use orphan branches                | Prevent configuration conflicts |
| YAML Validation       | Run `yamllint` before commits      | Avoid syntax errors             |
| Protected Branches    | Enable branch protection rules     | Prevent direct modifications    |
| Separate Environments | Create staging/production branches | Test changes safely             |
| Regular Audits        | Review commit history monthly      | Maintain compliance             |

## Troubleshooting Guide

**Q:** Why aren't DNS changes appearing in Cloudflare?  
**A:** Verify:

1. Webhook URL matches branch name in Semaphore
2. Ansible playbook has valid Cloudflare credentials
3. No YAML syntax errors in `records.yaml`

**Common Errors**:
| Error Message | Solution |
|--------------|----------|
| "Invalid API token" | Update credentials in Semaphore |
| "Branch not found" | Verify orphan branch exists |
| "YAML parse error" | Run `yamllint` on configuration |
| "Permission denied" | Check branch protection settings |

**Debugging Workflow**:

1. Check Semaphore job logs for execution details
2. Validate YAML with:

```bash
yamllint records.yaml
```

3. Verify webhook payload delivery in Git provider UI

---
**THIS REPOSITORY IS ENCRYPTED. IF YOU'RE HERE, YOU'RE EITHER VERY BRAVE OR VERY LOST. EITHER WAY, GOOD LUCK!**
