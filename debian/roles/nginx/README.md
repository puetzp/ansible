nginx
=====

This role configures `nginx` on Debian servers. The package `nginx` is installed if necessary.

The directories `/etc/nginx/modules-available` and `/etc/nginx/modules-enabled` are not manageable on purpose, since in general packages install modules in `/usr/lib/nginx/modules` and enable them by creating a corresponding `.conf` file in `/etc/nginx/modules-enabled`. Thus enabling and disabling modules is the same as installing and removing packages containing those modules and there is no need to implement similar logic in this Ansible role.

The directory `/etc/nginx/snippets` is not manageable either, because snippets are less useful when configuration is managed by Ansible where you can take advantage of templating to achieve the same result.

Requirements
------------

None

Role Variables
--------------

- `nginx_conf`: String. If present, the contents of this variable are copied to `/etc/nginx/nginx.conf`. If absent, the current file is not modified.
- `nginx_conf_d`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/nginx/conf.d` while the value is used as file content. Unknown files are removed from the directory. If absent, the directory is not modified.
- `nginx_sites_available`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/nginx/sites-available` while the value is used as file content. Existing files are left unmodified. If absent, the directory is not modified.
- `nginx_sites_enabled`: List. If present, every item from this list is used to create a symlink in `/etc/nginx/sites-enabled` pointing the corresponding file in `/etc/nginx/sites-available`. Unknown symlinks and files are removed from the directory. If absent, the directory is not modified.
- `nginx_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/nginx.service.d/override.conf` to override or add settings to the systemd service unit running `nginx`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  tasks:
    - name: Create directory /etc/nginx/tls
      ansible.builtin.file:
        path: /etc/nginx/tls
        state: directory
        owner: root
        group: root
        mode: '0755'
  roles:
    - ansible.debian.nginx
```

Example variables
-----------------

```yml
# host_vars/example.internal.yml

nginx_conf: "{{ lookup('file', 'etc/nginx/nginx.conf', rstrip = false) }}"

nginx_conf_d: {}

nginx_sites_available:
  example: |
    server {
      listen 443 ssl default_server;

      keepalive_timeout   70;

      ssl_certificate     /etc/nginx/tls/server.crt;
      ssl_certificate_key /etc/nginx/tls/server.key;
      ssl_session_cache   shared:SSL:10m;
      ssl_session_timeout 10m;

      root /var/www/html;

      index index.html index.htm index.nginx-debian.html;

      server_name {{ inventory_hostname }};

      location / {
        try_files $uri $uri/ =404;
      }
    }

nginx_sites_enabled:
  - example

nginx_systemd_override: |
  [Unit]
  ConditionPathExists=/etc/nginx/tls/server.crt
```

Links
-----

- [nginx(8)](https://manpages.debian.org/latest/nginx/nginx.8.en.html)
- [documentation](https://nginx.org/en/docs/)
