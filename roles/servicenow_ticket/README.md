<!-- This was created with Claude Code -->

servicenow_ticket
=================

An Ansible role for servicenow ticket.

Requirements
------------

- Ansible 2.9 or higher
- Access to target systems with appropriate permissions

Role Variables
--------------

* **servicenow_ticket**: Variable used in: Create ticket
  - Default: `create`

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: all
  roles:
    - role: servicenow_ticket
      vars:
        servicenow_ticket: <value>
```

License
-------

GNU General Public License v3.0 or later

Author Information
------------------

Red Hat Ansible Automation Platform
