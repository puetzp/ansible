hcloud_vm
=========

This role uses the Hetzner Cloud (hcloud) rescue system to install a fresh Debian 13 (trixie) operating system on the primary disk of a new virtual machine (VM). In this setup no preconfigured disk images (e.g. Cloud Images) are needed.

At the time of writing the rescue system runs Debian 12 (bookworm) in-memory and serves as a rich installation environment featuring all necessary tools to bootstrap Debian 13 on the primary disk using [`debootstrap`](https://manpages.debian.org/trixie/debootstrap/debootstrap.8.en.html). Hetzner also provides [package mirrors](https://docs.hetzner.com/robot/dedicated-server/operating-systems/hetzner-package-mirror) that are used during installation.

Since the role installs a fresh Debian 13 OS inside the rescue system of a new VM, it customizes the new environment and sets up networking, the correct hostname, WireGuard etc. Since these steps happen during installation, no `cloud-init` is needed to customize the system on first boot.

New VMs are created with a IPv6 address only. The role configures a WireGuard interface using the IPv6 address as endpoint. Inside the tunnel IPv4 is used. By default the new host's SSH server will only be accessible via WireGuard after rebooting.

> As you can see the role is quite opinionated and *not designed to be general-purpose*.

The role configures:

- LVM on the secondary partition to use distinct mount points for some directories (e.g. `/var`)
- `ifupdown` for networking which assigns the VMs IPv6 address to the primary interface
- a `wg0` interface for WireGuard using a private IPv4 address
- a hostname and correct `/etc/hosts` using the private IPv4 address
- `systemd-resolved` as stub resolver using Quad9 nameservers via TLS
- Hetzner package mirrors in `/etc/apt/sources.list`
- secure `sshd_config` listening on the `wg0` IPv4 address only

The SSH host keys of the new host as well as the WireGuard public key are added to a cache directory on the deployment server. The [`wireguard`](../wireguard) and [`ssh`](../ssh) roles can be used to configure WireGuard and SSH on the deployment server to enable connectivity to the new Debian host via WireGuard and by circumventing the trust-on-first-use problem with SSH.

This role runs Ansible tasks on two types of nodes:

- the deployment server (`localhost`) that initiates the play
- the rescue system of a fresh VM that is created by the role and is used to bootstrap Debian 13 on the primary disk

Preparation
-----------

Install `hcloud-cli` as described in [setting up hcloud](https://github.com/hetznercloud/cli/blob/main/docs/tutorials/setup-hcloud-cli.md).

The context is set up by the Ansible role. You will have to create an Access Token however and pass it to Ansible via the inventory (see below).

VM Installation
---------------

The Ansible role runs in the context of the new host, but performs some tasks on `localhost` via `delegate_to`. The following example configuration assumes that your playbooks are located in `$HOME/ansible`.

```yml
# $HOME/ansible/host_vars/dns.hcloud.internal.yml
---
ansible_host: 172.16.1.2

hcloud_vm_type: cx23
hcloud_vm_location: nbg1
```

```yml
# $HOME/ansible/group_vars/hcloud.yml
---
# Context used by hcloud-cli in which to interact with the API.
hcloud_vm_context: default

# Token associated with this context used to authenticate to the API.
# Use `--ask-vault-pass` with `ansible-playbook` if you choose to
# encrypt this variable.
hcloud_vm_token: !vault |
  <encrypted string>

# The name of the SSH key that you added to your hcloud account.
# This is injected into the rescue system to bootstrap the VM via
# Ansible.
hcloud_vm_ssh_key_name: me

# Your SSH public key to be injected into the bootstrapped VM.
hcloud_vm_ssh_key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"

# Encrypted password for root. For example this is the hash of
# the string `foobar`, generated via `openssl passwd -6`.
# Nowadays this should ideally be a salted yescrypt hash that is
# encrypted with `ansible-vault`.
hcloud_vm_root_password: $6$E9akWI2zpi.biv1g$YHGfQ2dzfmFsAbEpX/j7t22K5qqJTevm3XfslT4GiI9GYdsE8lZeFWpgal8D66sLr.r2Y/tppnhFNxFTc4o7V.

# The IP address and public key of your WireGuard client
# on the deployment server that runs the playbook. This will
# be configured as the sole peer on the new host.
hcloud_vm_wireguard_allowed_ip: 172.16.1.1/32
hcloud_vm_wireguard_public_key: <string>
```

Use the following example playbook to install the VMs:

```yml
# $HOME/ansible/deploy.yml
---
- hosts: hcloud
  gather_facts: false
  roles:
    - role: ansible.debian.hcloud_vm
```

```sh
ansible-playbook deploy.yml
```

Inspiration
-----------

- [FAI](https://fai-project.org/)
- [How to setup the Debian Linux image from scratch](https://mvallim.github.io/kubernetes-under-the-hood/documentation/create-linux-image.html)

