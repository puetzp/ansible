lvm
===

This role configures LVM physical volumes, volume groups, logical volumes and filesystems on logical volumes. In fact it is just a thin wrapper around modules from the `community.general` collection.

Requirements
------------

- `community.general` >= 13.4.0

Role Variables
--------------

- `lvm_pvs`: List. If present, configures each device as a physical volume.
- `lvm_vgs`: List. If present, creates volume groups using specific physical volumes.
- `lvm_lvs`: List. If present, creates logical volumes in a specific volume group and creates a filesystem in the new logical volume.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: all
  gather_facts: false
  roles:
    - ansible.debian.lvm
```

Example variables
-----------------

```yml
---
lvm_pvs:
  - /dev/vdb

lvm_vgs:
  - name: data
    pvs:
      - /dev/vdb

lvm_lvs:
  - name: mirror
    vg: data
    size: 100%FREE
    filesystem: ext4
```
