Role Name
=========

This role configures `nftables` on Debian servers. The package `nftables` is installed if necessary and conflicting packages are removed.

Requirements
------------

None

Role Variables
--------------

- `nftables_conf`: String. If present, the contents of this variable are copied to `/etc/nftables.conf`. If absent, the current file is not modified.
- `nftables_conf_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/nftables.d` while the value is used as file content. This is useful to split up input, output and forward chains and keep a reasonable default in the main configuration file. Unknown files are removed from this directory. If absent, the directory is not modified.


Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.nftables
```

Example variables
-----------------

```yml
# group_vars/all.yml

nftables_conf: |
  #!/usr/sbin/nft -f

  flush ruleset

  table inet filter {
      chain input {
          type filter hook input priority 0; policy drop;
          iif "lo" accept
          ip saddr 127.0.0.0/8 counter packets 0 bytes 0 drop
          ip6 saddr ::1 counter packets 0 bytes 0 drop
          ip protocol tcp ct state established accept
          ip protocol udp ct state established accept
          tcp dport 22 accept
          include "/etc/nftables.d/input*"
      }

      chain forward {
          type filter hook forward priority 0; policy drop;
          include "/etc/nftables.d/forward*"
      }

      chain output {
          type filter hook output priority 0; policy drop;
          ip protocol tcp ct state established,related,new accept
          ip protocol udp ct state established,related,new accept
          include "/etc/nftables.d/output*"
      }
  }
```

```yml
# host_vars/server.internal.yml

# Define additional rules.
nftables_conf_d:
  input_https: |
    tcp dport 443 accept

# Or explicitly remove unknown rule files.
nftables_conf_d: {}
```
