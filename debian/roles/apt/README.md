Role Name
=========

This role configures `apt` on Debian servers.

Requirements
------------

None

Role Variables
--------------

- `apt_sources`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/apt/sources.list.d` while the value is used as file content. Note that keys in this mapping should end with `.list` or `.sources`. Unknown files in this directory are removed, as is the default file `/etc/apt/sources.list`. If absent, the directory is not modified.
- `apt_keyrings`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/apt/keyrings`. The value is a source path from where the keyring should be copied to the remote host. Unknown files are removed from the target directory. If absent, the directory is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.apt
```

Example variables
-----------------

```yml
# group_vars/all.yml

apt_sources:
  debian.sources: |
    Types: deb
    URIs: https://deb.debian.org/debian
    Suites: trixie trixie-updates
    Components: main
    Architectures: amd64
    Signed-By: /etc/apt/keyrings/debian-archive-trixie-automatic.gpg

    Types: deb
    URIs: https://security.debian.org/debian-security
    Suites: trixie-security
    Components: main
    Architectures: amd64
    Signed-By: /etc/apt/keyrings/debian-archive-trixie-security-automatic.gpg

apt_keyrings:
  debian-archive-trixie-automatic.gpg: /usr/share/keyrings/debian-archive-trixie-automatic.gpg
  debian-archive-trixie-security-automatic.gpg: /usr/share/keyrings/debian-archive-trixie-security-automatic.gpg

```
