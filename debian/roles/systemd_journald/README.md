systemd_journald
================

This role configures `systemd-journald` and restarts the service when the configuration changes.

Requirements
------------

None

Role Variables
--------------

- `systemd_journald`: String. If present, the contents of this variable are copied to the configuration file at `/etc/systemd/journald.conf`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.systemd_journald
```

Example variables
-----------------

```yml
# group_vars/all.yml

systemd_journald: |
  [Journal]
  Storage=auto
  Compress=yes
  Seal=yes
  SplitMode=uid
  SyncIntervalSec=5m
  RateLimitIntervalSec=30s
  RateLimitBurst=10000
  SystemMaxUse=40M
  SystemKeepFree=100M
  SystemMaxFileSize=5M
  SystemMaxFiles=100
  RuntimeMaxUse=40M
  RuntimeKeepFree=100M
  RuntimeMaxFileSize=5M
  RuntimeMaxFiles=100
  MaxRetentionSec=1month
  MaxFileSec=1month
  ForwardToSyslog=no
  ForwardToKMsg=no
  ForwardToConsole=no
  ForwardToWall=yes
  TTYPath=/dev/console
  MaxLevelStore=debug
  MaxLevelSyslog=debug
  MaxLevelKMsg=notice
  MaxLevelConsole=info
  MaxLevelWall=emerg
  MaxLevelSocket=debug
  LineMax=48K
  ReadKMsg=yes
  Audit=yes
```

Limitations
-----------

Journald namespaces are not supported.

Links
-----

- [systemd-journald(8)](https://manpages.debian.org/latest/systemd/systemd-journald.8.en.html)
- [journald.conf(5)](https://manpages.debian.org/latest/systemd/journald.conf.5.en.html)
