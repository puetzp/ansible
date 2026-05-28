wireguard
=========

This role configures `wireguard` on Debian servers. The package `wireguard` is installed if necessary. It also creates a public/private key pair in `/etc/wireguard` when no private key has been generated yet.

The role is currently focused on managing a single WireGuard interface, namely `wg0`. This should cover most use cases, but can become limiting when the role is used to configure a WireGuard jumphost/router having (possibly) multiple WireGuard interfaces.

Requirements
------------

None

Role Variables
--------------

- `wireguard_wg0_conf_path`: String. This variable tells the role where to put the interface configuration file (see [defaults](defaults/main.yml)).
- `wireguard_wg0_conf`: String. If present, the contents of this variable are copied to the path specified by `wireguard_wg0_conf_path`. If absent, the current file is not modified.
- `wireguard_systemd_override`: String. If present, the contents of this variable are copied to `/etc/systemd/system/wg-quick@wg0.service.d/override.conf` to override or add settings to the systemd service unit that manages the interface `wg0`. If absent, the service unit is left unmodified.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: servers
  gather_facts: false
  roles:
    - ansible.debian.wireguard
```

Example variables
-----------------

In this example it is assumed that there is a kind of WireGuard cache containing the public keys, endpoints and IP addresses of peers. In this example the file is called `wireguard.json`:

```json
{
  "wireguard_all_peers": {
    "some-peer.internal": {
      "pubkey": "yKJl0hzt1NB6zK4wUVGI9l5M+p654noQCebaFgAU/AU=",
      "endpoint": "[fd37:3da9:60e6::1]:51820",
      "allowed_ip": "172.16.1.2/32"
    },
    "another-peer.internal": {
      "pubkey": "R+D+W2OJB5csiS7VuLHvyhsa9G6kNr3QAOyYw9Fgh1A=",
      "endpoint": "[fded:62b6:5f9d::1]:51820",
      "allowed_ip": "172.16.1.3/32"
    }
  }
}
```

The following variables ensure that `server.internal` has a single peer `some-peer.internal`. The endpoint from the cache is ignored because the peer is expected to initiate the connection and endpoint information is derived automatically by WireGuard.

```yml
# host_vars/server.internal.yml
---
wireguard_clients:
  - some-peer.internal

wireguard_wg0_conf: |
  [Interface]
  Address = 172.16.1.50/32
  ListenPort = 51820
  PostUp = wg set %i private-key <(cat /etc/wireguard/privatekey)

  {% for peer, config in wireguard_all_peers.items() %}
  {% if peer in wireguard_clients %}
  [Peer]
  PublicKey = {{ config.pubkey }}
  AllowedIPs = {{ config.allowed_ip }}
  {% endif %}
  {% endfor %}
```

Assuming the WireGuard cache is a file in the current directory, the following command can be used to execute the playbook:

```sh
ansible-playbook --extra-vars @wireguard.json <playbook>
```

When there is another source of peers that are updated by a different mechanism, the other role variables can be used to concatenate both sources and write the result to `wg0.conf` before restarting the unit:

```yml
# host_vars/server.internal.yml
---

# additional variables
wireguard_wg0_conf_path: /etc/wireguard/wg0.conf.part

wireguard_systemd_override: |
  [Service]
  ExecStartPre=/bin/sh -c 'cat /etc/wireguard/wg0.conf.part /var/lib/wireguard/peers.conf > /etc/wireguard/wg0.conf'
```

Links
-----

- [WireGuard](https://www.wireguard.com/)
- [wg(8)](https://manpages.debian.org/latest/wireguard-tools/wg.8.en.html)
- [wg-quick(8)](https://manpages.debian.org/latest/wireguard-tools/wg-quick.8.en.html)
