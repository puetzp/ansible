systemd_resolved
=========

This role configures `systemd-resolved` as a stub resolver on Debian servers. The package `systemd-resolved` is installed if necessary. It also ensures that `/etc/resolv.conf` is a symlink pointing to `/run/systemd/resolve/stub-resolv.conf`.

Requirements
------------

None

Role Variables
--------------

- `systemd_resolved`: String. If present, the contents of this variable are copied to the configuration file at `/etc/systemd/resolved.conf`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.systemd_resolved
```

Example variables
-----------------

```yml
# group_vars/all.yml

systemd_resolved: |
  [Resolve]
  DNS=192.168.1.100 192.168.1.101
  LLMNR=false

```


Links
-----

- [systemd-resolved(8)](https://manpages.debian.org/latest/systemd-resolved/systemd-resolved.8.en.html)
- [resolved.conf(5)](https://manpages.debian.org/latest/systemd-resolved/resolved.conf.5.en.html)
