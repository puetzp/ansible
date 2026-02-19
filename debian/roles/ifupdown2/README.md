Role Name
=========

This role configures `ifupdown2` for networking on Debian servers. The package `ifupdown2` is installed if necessary and conflicting packages are removed.

Requirements
------------

None

Role Variables
--------------

- `ifupdown2_config`: String. If present, the contents of this variable are copied to `/etc/network/ifupdown2/ifupdown2.conf`. If absent, the current file is not modified.
- `ifupdown2_addons`: String. If present, the contents of this variable are copied to `/etc/network/ifupdown2/addons.conf`. If absent, the current file is not modified.
- `ifupdown2_defaults`: String. If present, the contents of this variable are copied to `/etc/default/networking`. If absent, the current file is not modified.
- `ifupdown2_interfaces`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/network/interfaces.d` while the value is used as file content. Unknown files are removed from the directory. If absent, the directory is not modified.
- `ifupdown2_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/networking.service.d/override.conf` to override or add settings to the systemd service unit `networking`. If absent, the current file is not modified.


Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.ifupdown2
```

Example variables
-----------------

```yml
# group_vars/all.yml

# A little fix to ensure that services that depend on specific
# interfaces being up do not immediately fail to start on boot
# due to a missing network device.
ifupdown2_systemd_override: |
  [Unit]
  After=systemd-udevd.service
  Before=shutdown.target network.target network-online.target

```

```yml
# host_vars/server.internal.yml

ifupdown2_interfaces:
  enp1s0: |
    auto enp1s0
    iface enp1s0 inet static
      address 192.168.1.100
      netmask 255.255.255.0
      gateway 192.168.1.1

```

Caveats
-------

The contents of the file `/etc/network/interfaces` are predefined as follows:

```
source /etc/network/interfaces.d/*
```

Also the loopback interface (`/etc/network/interfaces.d/lo`) is silently added to the list of interfaces and will always be created, even when the `ifupdown2_interfaces` mapping is intentionally left empty.

Also note that some aspects of this role, such as the `networking` systemd service unit are not unique to `ifupdown2`. Still `ifupdown2` is always used as variable prefix to ensure consistency in the naming scheme.
