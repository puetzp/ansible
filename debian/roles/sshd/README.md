Role Name
=========

This role configures `sshd` on Debian servers.

Requirements
------------

None

Role Variables
--------------

- `sshd_config`: String. If present, the contents of this variable are copied to the configuration file at `/etc/ssh/sshd_config`. If absent, the current file is not modified.
- `sshd_config_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/ssh/sshd_config.d` while the value is used as file content. This is particularly useful to define additional `Match` blocks. Unknown files are removed from the directory. If absent, the directory is not modified.
- `sshd_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/sshd.service.d/override.conf` to override or add settings to the systemd service unit running `sshd`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.sshd
```

Example variables
-----------------

```yml
# group_vars/all.yml

# Mostly static configuration file, but used as a template to
# substitute ansible_host for ListenAddress.
sshd_config: |
  AcceptEnv LANG LC_*
  AddressFamily inet
  AllowAgentForwarding no
  AllowStreamLocalForwarding no
  AllowTcpForwarding no
  AllowUsers ansible
  AuthenticationMethods publickey,password
  AuthorizedKeysFile .ssh/authorized_keys
  Banner /etc/issue.net
  Ciphers aes128-ctr,aes192-ctr,aes256-ctr
  ClientAliveCountMax 3
  ClientAliveInterval 20
  Compression no
  DebianBanner no
  DisableForwarding yes
  GatewayPorts no
  GSSAPIAuthentication no
  HostbasedAuthentication no
  HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521
  IgnoreRhosts yes
  KexAlgorithms ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256
  ListenAddress {{ ansible_host }}
  LoginGraceTime 10
  LogLevel VERBOSE
  MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1
  MaxAuthTries 4
  MaxSessions 10
  MaxStartups 5
  ModuliFile /etc/ssh/moduli
  PasswordAuthentication yes
  PermitEmptyPasswords no
  PermitRootLogin no
  PermitUserEnvironment no
  PidFile /run/sshd.pid
  Port 22
  PrintLastLog no
  PrintMotd no
  PubkeyAcceptedAlgorithms ssh-ed25519
  PubkeyAuthentication yes
  StrictModes yes
  Subsystem sftp /usr/lib/openssh/sftp-server
  UseDNS no
  UsePAM yes
  X11Forwarding no
  Include /etc/ssh/sshd_config.d/*.conf

# Ensure no other files are included by removing unknown
# files from the directory.
sshd_config_d: {}
```
