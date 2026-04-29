<!-- This was created with Claude Code -->

# SNOW Automation

[![Contribute](https://img.shields.io/badge/OpenShift-Dev%20Spaces-525C86?logo=redhatopenshift&labelColor=EE0000)](https://devspaces.apps.ocp.shadowman.dev/#https://github.com/shadowman-lab/Ansible-SNOW)

This directory contains Ansible automation for snow management and operations.

## Overview

The SNOW automation provides playbooks and roles for managing and configuring
snow infrastructure and services.

## Roles

| Role | Description |
| ---- | ----------- |
| [servicenow_catalog](roles/servicenow_catalog/README.md) | Role for servicenow catalog |
| [servicenow_catalog_create](roles/servicenow_catalog_create/README.md) | Role for servicenow catalog create |
| [servicenow_change_request](roles/servicenow_change_request/README.md) | Role for servicenow change request |
| [servicenow_change_request_schedule](roles/servicenow_change_request_schedule/README.md) | Role for servicenow change request schedule |
| [servicenow_change_request_wait](roles/servicenow_change_request_wait/README.md) | Role for servicenow change request wait |
| [servicenow_cmdb](roles/servicenow_cmdb/README.md) | Role for servicenow cmdb |
| [servicenow_ritm_retrieve_eda](roles/servicenow_ritm_retrieve_eda/README.md) | Role for servicenow ritm retrieve eda |
| [servicenow_ticket](roles/servicenow_ticket/README.md) | Role for servicenow ticket |
| [servicenow_user](roles/servicenow_user/README.md) | Role for servicenow user |

## Playbooks

| Playbook | Description | Target Hosts |
| -------- | ----------- | ------------ |
| ServiceNowCR_and_approve.yml | Playbook for ServiceNowCR and approve | localhost |
| ServiceNowCR_canceled.yml | Playbook for ServiceNowCR canceled | localhost |
| ServiceNowCR_closed.yml | Playbook for ServiceNowCR closed | localhost |
| ServiceNowCR_create.yml | Playbook for ServiceNowCR create | all |
| ServiceNowCR_deleted.yml | Playbook for ServiceNowCR deleted | localhost |
| ServiceNowCR_implement.yml | Playbook for ServiceNowCR implement | localhost |
| ServiceNowCatalog.yml | Playbook for ServiceNowCatalog | localhost |
| ServiceNowCatalogCreate.yml | Playbook for ServiceNowCatalogCreate | localhost |
| ServiceNowEDARITM.yml | Playbook for ServiceNowEDARITM | localhost |
| ServiceNow_change_request_schedule.yml | Playbook for ServiceNow change request schedule | localhost |
| ServiceNow_cmdb_update.yml | Playbook for ServiceNow cmdb update | {{ vm_name | default('all') }} |
| ServiceNowticket.yml | Playbook for ServiceNowticket | {{ vm_name | default('all')}} |
| ServiceNowticket_close.yml | Playbook for ServiceNowticket close | all |
| ServiceNowticket_eda.yml | Playbook for ServiceNowticket eda | {{ vm_name | default('all')}} |
| ServiceNowticket_find.yml | Playbook for ServiceNowticket find | localhost |
| ServiceNowticket_inprogress.yml | Playbook for ServiceNowticket inprogress | all |
| ServiceNowticket_logs.yml | Playbook for ServiceNowticket logs | {{ vm_name | default('all')}} |
| ServiceNowticket_logs_update.yml | Playbook for ServiceNowticket logs update | {{ vm_name | default('all')}} |
| ServiceNowticket_security.yml | Playbook for ServiceNowticket security | localhost |
| ServiceNowuser_create.yml | Playbook for ServiceNowuser create | localhost |

## Usage

### Running with ansible-navigator

```bash
# Run a playbook
ansible-navigator run ServiceNowCR_and_approve.yml

# Run in stdout mode
ansible-navigator run ServiceNowCR_and_approve.yml -m stdout
```

### Using roles

```yaml
- hosts: target_hosts
  roles:
    - servicenow_catalog
```

## Requirements

- Ansible 2.9 or higher (via ansible-navigator)
- Required collections (see `collections/requirements.yml` if present)
- Appropriate access credentials configured via environment variables

## Directory Structure

```
Ansible-SNOW/
├── roles/              # Ansible roles
├── *.yml               # Playbooks
├── collections/        # Collection dependencies (if present)
└── ansible-navigator.yml  # Navigator configuration
```
