Role Name
=========

This role configures `/etc/hosts` on Debian servers.

Requirements
------------

None

Role Variables
--------------

- `hosts`: String. If present, the contents of this variable are copied to `/etc/hosts`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.hosts
```

Example variables
-----------------

```yml
# group_vars/all.yml

# This configuration expects that the hosts in your inventory
# are defined with their FQDN.
hosts: |
  {{ ansible_host }} {{ inventory_hostname }} {{ inventory_hostname_short }}

```
