sysctl
=====

This role configures kernel parameters on Debian servers via `systemd-sysctl`.

Requirements
------------

None

Role Variables
--------------

- `sysctl_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/sysctl.d` while the value is used as file content. Note that keys in this mapping should end with `.conf`, otherwise they are ignored by `systemd-sysctl`. Unknown files in this directory are removed, as is the obsolete default file `/etc/sysctl.conf`. If absent, files in the directory are not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.sysctl
```

Example variables
-----------------

```yml
# group_vars/all.yml

sysctl_d:
  hardening.conf: |
    kernel.randomize_va_space = 2
    kernel.yama.ptrace_scope = 2
    fs.suid_dumpable = 0
  network.conf: |
    net.ipv4.ip_forward = 0
    net.ipv6.conf.all.forwarding = 0
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0
    net.ipv4.icmp_ignore_bogus_error_responses = 1
    net.ipv4.icmp_echo_ignore_broadcasts = 1
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv6.conf.all.accept_redirects = 0
    net.ipv6.conf.default.accept_redirects = 0
    net.ipv4.conf.all.secure_redirects = 0
    net.ipv4.conf.default.secure_redirects = 0
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv6.conf.all.accept_source_route = 0
    net.ipv6.conf.default.accept_source_route = 0
    net.ipv4.conf.all.log_martians = 1
    net.ipv4.conf.default.log_martians = 1
    net.ipv4.tcp_syncookies = 1
    net.ipv6.conf.all.accept_ra = 0
    net.ipv6.conf.default.accept_ra = 0
```

Links
-----

- [systemd-sysctl.service(8)](https://manpages.debian.org/latest/systemd/systemd-sysctl.service.8.en.html)
- [sysctl.d(5)](https://manpages.debian.org/latest/systemd/sysctl.d.5.en.html)
- [sysctl(8)](https://manpages.debian.org/latest/procps/sysctl.8.en.html)
