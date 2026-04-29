<!-- This was created with Claude Code -->

servicenow_change_request_schedule
==================================

An Ansible role for servicenow change request schedule.

Requirements
------------

- Ansible 2.9 or higher
- Access to target systems with appropriate permissions

Role Variables
--------------

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
    - role: servicenow_change_request_schedule
      vars:
        to_email: <value>
```

License
-------

GNU General Public License v3.0 or later

Author Information
------------------

Red Hat Ansible Automation Platform
