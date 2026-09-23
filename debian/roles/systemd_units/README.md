systemd_units
=============

This role configures systemd `mount`, `service` and `timer` units on Debian servers.

Requirements
------------

None

Role Variables
--------------

| Name | Type | Description |
| --- | --- | --- |
| `systemd_mounts` | Mapping | If present, each key in the mapping is used to create a file in `/etc/systemd/system`. The `.mount` filename extension is added automatically. Inside the mapping the value of the key `content` determines the file content. Beyond that the mapping can contain any parameters from [`ansible.builtin.systemd_service`](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/systemd_service_module.html) which are passed on to the task managing the unit.
| `systemd_services` | Mapping | If present, each key in the mapping is used to create a file in `/etc/systemd/system`. The `.service` filename extension is added automatically. Inside the mapping the value of the key `content` determines the file content. Beyond that the mapping can contain any parameters from [`ansible.builtin.systemd_service`](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/systemd_service_module.html) which are passed on to the task managing the unit.
| `systemd_timers` | Mapping | If present, each key in the mapping is used to create a file in `/etc/systemd/system`. The `.timer` filename extension is added automatically. Inside the mapping the value of the key `content` determines the file content. Beyond that the mapping can contain any parameters from [`ansible.builtin.systemd_service`](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/systemd_service_module.html) which are passed on to the task managing the unit.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: all
  gather_facts: false
  roles:
    - ansible.debian.systemd_units
```

Example variables
-----------------

```yml
systemd_mounts:
  srv-mirror:
    state: started
    enabled: true
    daemon_reload: true
    content: |
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

systemd_timers:
  reprepro-update:
    state: started
    enabled: true
    daemon_reload: true
    content: |
      [Unit]
      Description=Periodic mirror data sync
      After=network.target

      [Timer]
      OnBootSec=60
      OnCalendar=daily

      [Install]
      WantedBy=multi-user.target

systemd_services:
  reprepro-update:
    daemon_reload: true
    content: |
      [Unit]
      Description=Periodic mirror data sync

      [Service]
      Type=oneshot
      User=reprepro
      Group=reprepro
      ExecStart=reprepro -V --basedir /etc/reprepro update trixie
      ExecStart=reprepro -V --basedir /etc/reprepro update trixie-updates
      ExecStart=reprepro -V --basedir /etc/reprepro update trixie-security
```

Links
-----

- [mount(8)](https://manpages.debian.org/latest/mount/mount.8.en.html)
- [systemd.mount](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html)
- [systemd.service](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [systemd.timer](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html)
- [systemd.unit](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html)
- [systemd.exec](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)
