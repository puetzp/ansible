Role Name
=========

This role configures `prometheus` on Debian servers. The package `prometheus` is installed if necessary.

Requirements
------------

None

Role Variables
--------------

- `prometheus_config`: String. If present, the contents of this variable are copied to `/etc/prometheus/prometheus.yml`. If absent, the current file is not modified.
- `prometheus_rule_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/prometheus/rules.d` while the value is used as file content. This is useful to define recording and alerting rules outside of the main configuration file. Unknown files are removed from the directory. If absent, the directory is not modified.
- `prometheus_scrape_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/prometheus/scrape.d` while the value is used as file content. This is useful to define scrape configurations outside of the main configuration file. Unknown files are removed from the directory. If absent, the directory is not modified.
- `prometheus_defaults`: String. If present, the contents of this variable are copied to `/etc/default/prometheus`. If absent, the current file is not modified.
- `prometheus_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/prometheus.service.d/override.conf` to override or add settings to the systemd service unit running `prometheus`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.prometheus
```

Example variables
-----------------

```yml
# host_vars/server.internal.yml

prometheus_defaults: |
  ARGS="--web.listen-address=192.168.1.100:9090 "

prometheus_config: "{{ lookup('file', 'files/etc/prometheus/prometheus.yml', rstrip = false) }}"

prometheus_scrape_d:
  node_exporter.yml: |
    scrape_configs:
      - job_name: node
        static_configs:
          - targets: ['localhost:9100']

prometheus_rule_d:
  http_total.yml: |
    groups:
      - name: example
        rules:
          - record: code:prometheus_http_requests_total:sum
            expr: sum by (code) (prometheus_http_requests_total)

```

Caveats
-------

Since Prometheus configuration files are written in YAML (or optionally JSON) it would make sense to write the values from `prometheus_rule_d` and `prometheus_scrape_d` to a file using the `to_yaml` filter. However the values in these mappings might be static file lookups (see the example above). Since the result of these lookups is a string, the `to_yaml` filter behaves not as expected and writes the result as a single quoted YAML scalar.

This is avoided by having all variables defined as (multi-line) strings, write it as-is to the destination file and leave it to `promtool` to validate the YAML syntax and configuration parameters inside.
