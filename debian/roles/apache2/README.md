apache2
=========

This role configures `apache2` on Debian servers. The package `apache2` is installed if necessary.

On Debian `apache2` pulls in configuration from the `/etc/apache2/*-available` directories by creating symlinks in the corresponding `/etc/apache2/*-enabled` directories. Since configuration snippets, modules and sites that are not explicitly enabled by a symlink are ignored by `apache2`, this role does not remove unknown, unmanaged files from the `/etc/apache2/*-available` directories. However the `/etc/apache2/*-enabled` directories are ensured to contain only symlinks that were defined in Ansible variables and any other symlinks and files are removed from those directories.

The `a2enmod` script and other common utilities are not used, because they make it harder to manage the configuration declaratively and its more convenient to maintain the state of the `/etc/apache2/*-enabled` directories by creating the symlinks via Ansible.

Requirements
------------

None

Role Variables
--------------

- `apache2_conf`: String. If present, the contents of this variable are copied to `/etc/apache2/apache2.conf`. If absent, the current file is not modified.
- `apache2_ports`: String. If present, the contents of this variable are copied to `/etc/apache2/ports.conf`. If absent, the current file is not modified.
- `apache2_conf_available`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/apache2/conf-available` while the value is used as file content. Existing files are left unmodified. If absent, the directory is not modified.
- `apache2_conf_enabled`: List. If present, every item from this list is used to create a symlink in `/etc/apache2/conf-enabled` pointing to the corresponding file in `/etc/apache2/conf-available`. Unknown symlinks and files are removed from the directory. If absent, the directory is not modified.
- `apache2_mods_available`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/apache2/mods-available` while the value is used as file content. Existing files are left unmodified. If absent, the directory is not modified.
- `apache2_mods_enabled`: List. If present, every item from this list is used to create a symlink in `/etc/apache2/mods-enabled` pointing the corresponding file in `/etc/apache2/mods-available`. Unknown symlinks and files are removed from the directory. If absent, the directory is not modified.
- `apache2_sites_available`: Mapping. If present, each key in this mapping is used as a file name to create a file in `/etc/apache2/sites-available` while the value is used as file content. Existing files are left unmodified. If absent, the directory is not modified.
- `apache2_sites_enabled`: List. If present, every item from this list is used to create a symlink in `/etc/apache2/sites-enabled` pointing the corresponding file in `/etc/apache2/sites-available`. Unknown symlinks and files are removed from the directory. If absent, the directory is not modified.
- `apache2_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/apache2.service.d/override.conf` to override or add settings to the systemd service unit running `apache2`. If absent, the current file is not modified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.apache2
```

Example variables
-----------------

```yml
# host_vars/mirror.internal.yml

apache2_conf: "{{ lookup('file', 'etc/apache2/apache2.conf', rstrip = false) }}"

apache2_ports: |
  Listen {{ ansible_host }}:80

apache2_mods_enabled:
  - access_compat.load
  - alias.conf
  - alias.load
  - authz_core.load
  - authz_host.load
  - autoindex.conf
  - autoindex.load
  - deflate.conf
  - deflate.load
  - dir.conf
  - dir.load
  - env.load
  - filter.load
  - mime.conf
  - mime.load
  - mpm_event.conf
  - mpm_event.load
  - negotiation.conf
  - negotiation.load
  - reqtimeout.conf
  - reqtimeout.load
  - setenvif.conf
  - setenvif.load
  - status.conf
  - status.load

apache2_conf_enabled:
  - security.conf

apache2_sites_available:
  mirror.conf: |
    <VirtualHost *:80>
      ServerName www.example.com

      DocumentRoot /var/www/mirror

      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>


apache2_sites_enabled:
  - mirror.conf
```

Links
-----

- [apache2(8)](https://manpages.debian.org/latest/apache2-bin/apache2.8.en.html)
- [documentation](https://httpd.apache.org/docs/current)
