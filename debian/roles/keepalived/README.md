keepalived
=========

This role configures `keepalived` on Debian servers. The package `keepalived` is installed if it does not exist.

Requirements
------------

None

Role Variables
--------------

- `keepalived_conf`: String. If present, the contents of this variable are copied to `/etc/keepalived/keepalived.conf`. If absent, the current file is not modified.
- `keepalived_conf_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/keepalived/conf.d` while the value is used as file content. Unknown files are removed from the directory. If absent, the directory is not modified.
- `keepalived_defaults`: String. If present, the contents of this variable are copied to `/etc/default/keepalived`. If absent, the current file is not modified.
- `keepalived_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/keepalived.service.d/override.conf` to override or add settings to the systemd service unit running `keepalived`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.keepalived
```

Example variables
-----------------

The following example configures three servers to act as a failover cluster for an Apache webserver backend.

```yml
# group_vars/all.yml
---

keepalived_defaults: |
  DAEMON_ARGS="--vrrp"

keepalived_conf_d:
  track.conf: |
    vrrp_track_process APACHE {
      process /usr/sbin/apache2
      param_match partial
      weight -254
      quorum 1
    }

keepalived_include: includer /etc/keepalived/conf.d/*

keepalived_passphrase: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  39626534306434393933393631363461633933623066363537346564306430393437363165656231
  6266353636613839373736376565643333653236386431360a663937633163363032323630653536
  30613761323465663730343737626638636665303065303061633765366130326137643161323534
  3133356136353538380a396332623333656239316638653436663831346663363038633966623238
  6134
```

```yml
# host_vars/test-1.internal.yml
---

keepalived_conf: |
  {{ keepalived_include }}

  vrrp_instance APACHE {
    state MASTER
    interface enp1s0
    virtual_router_id 107
    priority 150
    
    track_process {
      APACHE
    }

    unicast_peer {
      192.168.122.3
      192.168.122.4
    }

    authentication {
      auth_type PASS
      auth_pass {{ keepalived_passphrase }}
    }

    virtual_ipaddress {
      192.168.122.10/24 dev enp1s0
    }
  }

```

```yml
# host_vars/test-2.internal.yml
---

keepalived_conf: |
  {{ keepalived_include }}

  vrrp_instance APACHE {
    state BACKUP
    interface enp1s0
    virtual_router_id 107
    priority 100
    nopreempt

    track_process {
      APACHE
    }

    unicast_peer {
      192.168.122.2
      192.168.122.4
    }

    authentication {
      auth_type PASS
      auth_pass {{ keepalived_passphrase }}
    }

    virtual_ipaddress {
      192.168.122.10/24 dev enp1s0
    }
  }

```

```yml
# host_vars/test-3.internal.yml
---

keepalived_conf: |
  {{ keepalived_include }}

  vrrp_instance APACHE {
    state BACKUP
    interface enp1s0
    virtual_router_id 107
    priority 90
    nopreempt

    track_process {
      APACHE
    }

    unicast_peer {
      192.168.122.2
      192.168.122.3
    }

    authentication {
      auth_type PASS
      auth_pass {{ keepalived_passphrase }}
    }

    virtual_ipaddress {
      192.168.122.10/24 dev enp1s0
    }
  }

```

Links
-----

- [keepalived(8)](https://manpages.debian.org/latest/keepalived/keepalived.8.en.html)
- [keepalived.conf(5)](https://manpages.debian.org/latest/keepalived/keepalived.conf.5.en.html)
