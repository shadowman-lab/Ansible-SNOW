<!-- This was created with Claude Code -->

servicenow_change_request_wait
==============================

An Ansible role for servicenow change request wait.

Requirements
------------

- Ansible 2.9 or higher
- Access to target systems with appropriate permissions

Role Variables
--------------

* **cr_description**: Variable used in: Create ServiceNow Change Request
  - Default: `Create new {{ operating_system | default('') }} server in {{ shadowman_provision_hypervisor | default('production') }}`

* **to_email**: Email address
  - Default: `{{ to_emails.split(',') }}`

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: all
  roles:
    - role: servicenow_change_request_wait
      vars:
        cr_description: <value>
        to_email: <value>
```

License
-------

GNU General Public License v3.0 or later

Author Information
------------------

Red Hat Ansible Automation Platform
