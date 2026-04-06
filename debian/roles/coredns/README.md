coredns
=======

This role configures `coredns` on Debian servers. Since `coredns` is not packaged by Debian, the binary needs to be downloaded from GitHub. The role provides parameters to download a specific version, which is then symlinked to `/usr/local/bin/coredns`.

Requirements
------------

None

Role Variables
--------------

- `coredns_version`: String. The version of coredns that is to be downloaded from GitHub, e.g. `1.14.2` (without the leading "v"!). The role downloads the archive `coredns_{{ coredns_version }}_linux_amd64.tgz`.
- `coredns_checksum`: String. The checksum of the archive that is downloaded from GitHub, e.g. `sha256:2f08896df9d28ea0cd2294037e6d66e82f996b504e4901b791be8a3ae042029b`.
- `coredns_corefile`: String. If present, the contents of this variable are copied to `/etc/coredns/Corefile`. If absent, the current file is not modified.
- `coredns_defaults`: String. If present, the contents of this variable are copied to `/etc/default/coredns`. If absent, the role creates a commented example file that does not change the defaults.
- `coredns_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/coredns.service.d/override.conf` to override or add settings to the systemd service unit running `coredns`. If absent, the service unit is left unmodified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.coredns
```

Example variables
-----------------

```yml
# host_vars/dns.internal.yml
---

coredns_version: 1.14.2

coredns_checksum: sha256:2f08896df9d28ea0cd2294037e6d66e82f996b504e4901b791be8a3ae042029b

coredns_defaults: |
  OPTS=-p 1053

coredns_corefile: |
  . {
    bind 127.0.0.1
    chaos
  }
```

Links
-----

- [CoreDNS](https://coredns.io)
- [GitHub](https://github.com/coredns/coredns)
