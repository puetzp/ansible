Ansible collection for Debian
=============================

This is my personal Ansible collection for configuring Debian servers.

It is focused on including tried-and-tested configuration files into configuration management without having to translate various configuration languages/formats to Ansible variables. In fact the Ansible part is trivial as it mostly just copies file contents from the management server to remote hosts and relies on the user to template configuration at their leisure.

None of the roles in this collection use predefined files/templates. Files and templates are instead kept outside of the role logic, e.g. at the inventory level with other group and host variables.

In fact this pattern benefits immensely from the way Ansible handles variables. Variables can be anything from static values over string substitutions to complex templates as long as they eventually evaluate to a valid YAML type.

The roles in this collection predefine only a few variables that are needed to copy file contents from variables to their proper place in the destination file system and run handlers to enable the new configuration. Fortunately the Debian maintainers already define how the configuration for most packaged software should be handled. The roles in this collection only provide an interface to this structure.

Example
-------

The [sshd](debian/roles/sshd) offers only a few variables, most notably:

- `sshd_config`
- `sshd_config_d`

The first manages the contents of `/etc/ssh/sshd_config` while the latter creates files in `/etc/ssh/sshd_config.d` to be included by the main configuration file.

Now using Ansible variables there are multiple ways to define the contents of these files (shortened for brevity):

```yml
---
sshd_config: |
  AcceptEnv LANG LC_*
  AddressFamily inet
  ListenAddress {{ ansible_host }}
  Port 22
  PubkeyAuthentication yes
  PasswordAuthentication yes
  AuthenticationMethods publickey,password
  Include /etc/ssh/sshd_config.d/*.conf

sshd_config_d: {}
```

```yml
---
sshd_config: "{{ lookup('ansible.builtin.file', 'files/etc/ssh/sshd_config') }}"

sshd_config_d:
  ansible.conf: |
    Match User ansible
	  AuthenticationMethods publickey
```

```yml
---
sshd_config: "{{ lookup('ansible.builtin.template, 'files/etc/ssh/sshd_config.j2') }}"
```

```yml
---
sshd_config: |
  AcceptEnv LANG LC_*
  AddressFamily inet
  {% for ip in ips %}
  ListenAddress {{ ip }}
  {% endfor %}
  Port 22
  PubkeyAuthentication
  Include /etc/ssh/sshd_config.d/*.conf

```

Any combination of the methods above can be used to define configuration, from completely static and tested files that are centralized on the Ansible server to be rolled out to all servers, to complex templates that still only contain variables for the bits of the configuration that are indeed varied.
