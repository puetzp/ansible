systemd_mounts
==============

This role configures systemd `mount` units on Debian servers.

Requirements
------------

None

Role Variables
--------------

- `systemd_mounts`: Mapping. If present, each key in the mapping is used to create a file in `/etc/systemd/system` while the value is used as file content. The `.mount` filename extension is added automatically.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: all
  gather_facts: false
  roles:
    - ansible.debian.systemd_mounts
```

Example variables
-----------------

```yml
systemd_mounts:
  srv-mirror: |
    [Unit]
    Description=Debian mirror data mount

    [Mount]
    What=/dev/mapper/data-mirror
    Where=/srv/mirror
    Type=ext4
    Options=noatime,nosuid,noexec,nodev,rw
    TimeoutSec=3s

    [Install]
    WantedBy=multi-user.target
```

Links
-----

- [mount(8)](https://manpages.debian.org/latest/mount/mount.8.en.html)
- [systemd.unit](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html)
