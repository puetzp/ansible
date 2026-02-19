Role Name
=========

This role configures `chrony` on Debian servers. The package `chrony` is installed if necessary.

Requirements
------------

None

Role Variables
--------------

- `chrony_conf`: String. If present, the contents of this variable are copied to `/etc/chrony/chrony.conf`. If absent, the current file is not modified.
- `chrony_conf_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/chrony/conf.d` while the value is used as file content. Unknown files are removed from the directory. If absent, the directory is not modified.
- `chrony_sources_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/chrony/sources.d` while the value is used as file content. Unknown files are removed from the directory. If absent, the directory is not modified.
- `chrony_keys`: String. If present, the contents of this variable are copied to `/etc/chrony/chrony.keys`. If absent, the current file is not modified.
- `chrony_defaults`: String. If present, the contents of this variable are copied to `/etc/default/chrony`. If absent, the current file is not modified.
- `chrony_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/chrony.service.d/override.conf` to override or add settings to the systemd service unit running `chrony`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.chrony
```

Example variables
-----------------

```yml
# group_vars/all.yml

chrony_conf: |
  sourcedir /etc/chrony/sources.d
  keyfile /etc/chrony/chrony.keys
  driftfile /var/lib/chrony/chrony.drift
  ntsdumpdir /var/lib/chrony
  log tracking measurements statistics
  logdir /var/log/chrony
  maxupdateskew 100.0
  rtcsync
  makestep 1 3
  leapseclist /usr/share/zoneinfo/leap-seconds.list
  confdir /etc/chrony/conf.d

chrony_conf_d: {}

chrony_sources_d:
  debian.sources: |
    pool 2.debian.pool.ntp.org iburst

chrony_defaults: |
  DAEMON_OPTS="-F 1 -4"

```
