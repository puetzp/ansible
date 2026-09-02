ssh
===

At this point this role only configures the local `ssh` known hosts file of the current user on Debian servers. It also assumes that the SSH host keys are locally accessible in order to assemble the known hosts file.

Requirements
------------

None

Role Variables
--------------

- `ssh_known_hosts`: String. The contents of this variable are copied to the configuration file at `${HOME}/.ssh/known_hosts2`.

Dependencies
------------

None

Example Playbook
----------------

It is important that the playbook gather facts. Otherwise the role will not be able to determine the user it runs as.

```yml
- hosts: localhost 
  roles:
    - ansible.debian.ssh
```

The following example assumes that the role is used in conjunction with the [`hcloud_vm`](../hcloud_vm) role which caches SSH host keys in a child directory of the directory where the playbook is located.

```yml
---
ansible_connection: local

wireguard_peers:
  - example.internal

ssh_known_hosts: |
  {% for peer in wireguard_peers %}
  {{ peer }} {{ lookup('file', playbook_dir + '/cache/' + peer + '/ssh/ssh_host_ecdsa_key.pub') | trim }}
  {{ peer }} {{ lookup('file', playbook_dir + '/cache/' + peer + '/ssh/ssh_host_ed25519_key.pub') | trim }}
  {{ peer }} {{ lookup('file', playbook_dir + '/cache/' + peer + '/ssh/ssh_host_rsa_key.pub') | trim }}
  {% endfor %}
```

