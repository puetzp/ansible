prometheus_node_exporter
========================

This role configures `prometheus-node-exporter` on Debian servers. The package `prometheus-node-exporter` is installed if necessary.

Requirements
------------

None

Role Variables
--------------

- `prometheus_node_exporter_defaults`: String. If present, the contents of this variable are copied to `/etc/default/prometheus-node-exporter`. If absent, the current file is not modified.
- `prometheus_node_exporter_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/prometheus-node-exporter.service.d/override.conf` to override or add settings to the systemd service unit running `prometheus-node-exporter`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.prometheus_node_exporter
```

Example variables
-----------------

In this example it is assumed that `prometheus-node-exporter` is only reachable via WireGuard, to illustrate how to the exporter's configuration accordingly using role variables.

```yml
# group_vars/all.yml

prometheus_node_exporter_systemd_override: |
  [Unit]
  After=wg-quick@wg0.service
  Wants=wg-quick@wg0.service

prometheus_node_exporter_defaults: |
  ARGS="--web.listen-address={{ wireguard_host }}:9100 --collector.disable-defaults --collector.systemd --collector.meminfo --collector.loadavg --collector.systemd.unit-include=^(nftables|wg-quick@wg0|ssh|chrony|{{ prometheus_node_exporter_services | default([]) | join('|') }}).service"
```

```yml
# host_vars/server.internal.yml
---
wireguard_host: 10.10.10.3

# This is not a role variable, but a custom variable to further
# adjust settings on the host level that feeds into the templated
# group configuration.
prometheus_node_exporter_services:
  - coredns
```

Links
-----

- [prometheus-node-exporter(1)](https://manpages.debian.org/latest/prometheus-node-exporter/prometheus-node-exporter.1.en.html)
