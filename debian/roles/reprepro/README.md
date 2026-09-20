reprepro
========

This role configures `reprepro` to enable Debian repository management (mirroring or internal repositories).

The `reprepro` tool cannot simply mirror upstream repositories and keep the signatures on the release files. Instead `reprepro` downloads the contents of an upstream repository according to the [`conf/distributions`](https://manpages.debian.org/latest/reprepro/reprepro.1.en.html#conf/distributions) and [`conf/updates`](https://manpages.debian.org/latest/reprepro/reprepro.1.en.html#conf/updates) files and re-generates the release files and signs them with its own openPGP key.

This role takes care of generating an OpenPGP key pair and provides a script for signing releases (`conf/sign.sh`). The script provides access to the passphrase-protected secret via `systemd-creds`. Thus after configuring the passphrase for the private keyring once when setting up, `reprepro` operations can then be executed without entering the passphrase manually. See the examples below on how to use the signing script in a `conf/updates` file.

> In its current state the role does not execute `reprepro` commands such as `reprepro update <codename>` when a configuration file changes. The reasoning behind this is that such long-running operations should instead be executed in a background job, e.g. via systemd timers, path and service units.

Requirements
------------

None

Role Variables
--------------

- `reprepro_basedir`: String. The base directory where `reprepro` expects its configuration by default and where the signing script is created (defaul: `/etc/reprepro`).
- `reprepro_outdir`: String. The directory where `reprepro` creates repositories (default: `/srv/reprepro`).
- `reprepro_user`: String. The user account that executes `reprepro` commands and is able to access the private OpenPGP keyring via `systemd-creds` (default: `reprepro`).
- `reprepro_gpg_passphrase`: String. The passphrase to create and access the private OpenPGP keyring. This is also used to create a credential via `systemd-creds`, so the user is able to execute `reprepro` commands (such as `reprepro update <codename>`) automatically without human intervention.
- `reprepro_options`: String. If present, the contents of this variable are copied to `{{ reprepro_basedir }}/conf/options`.
- `reprepro_distributions`: String. If present, the contents of this variable are copied to `{{ reprepro_basedir }}/conf/distributions`.
- `reprepro_updates`: String. If present, the contents of this variable are copied to `{{ reprepro_basedir }}/conf/updates`.
- `reprepro_gpg_keyrings`: List. If present, imports all keys from local keyrings to the remote user's keyring. This way `reprepro` is able to validate upstream release files when refering to a key via the `VerifyRelease` directive in a `conf/updates` file.

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: mirror
  gather_facts: false
  roles:
    - ansible.debian.nginx
    - ansible.debian.reprepro
```

Example variables
-----------------

The following variables can be used to create configuration files to mirror current Debian upstream repositories.

```yml
---
reprepro_gpg_passphrase: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  64646132303635626533666266356333306635643734313338633662303533643739393066306663
  3539623537616265346338333633316339653564366563610a386632656233353738393430363566
  36623261393063616438326464393364356339376165316262313063333665626461346264616630
  6132323266626561320a326534396334316338323562613266653931656662376563633831663637
  3534

reprepro_gpg_keyrings:
  - /usr/share/keyrings/debian-archive-trixie-automatic.gpg
  - /usr/share/keyrings/debian-archive-trixie-security-automatic.gpg
 
reprepro_outdir: /srv/mirror/data

reprepro_options: |
  outdir {{ reprepro_outdir }}

# The `SignWith` directive can be used to refer to the signing script for
# signing releases when exporting.
reprepro_distributions: |
  Codename: trixie
  Origin: deb.debian.org
  Architectures: amd64
  Components: main
  Update: trixie
  SignWith: ! sign.sh

  Codename: trixie-updates
  Origin: deb.debian.org
  Architectures: amd64
  Components: main
  Update: trixie-updates
  SignWith: ! sign.sh

  Codename: trixie-security
  Origin: security.debian.org
  Architectures: amd64
  Components: main
  Update: trixie-security
  SignWith: ! sign.sh

reprepro_updates_filter_formula: >
  Section (!=cli-mono),
  Section (!=comm),
  Section (!=debian-installer),
  Section(!=debug),
  Section (!=education),
  Section (!=electronics),
  Section (!=embedded),
  Section (!=fonts),
  Section (!=games),
  Section (!=gnome),
  Section (!=gnu-r),
  Section (!=gnustep),
  Section (!=golang),
  Section (!=hamradio),
  Section (!=graphics),
  Section (!=haskell),
  Section (!=java),
  Section (!=javascript),
  Section (!=kde),
  Section (!=lisp),
  Section (!=mail),
  Section(!=news),
  Section (!=ocaml),
  Section (!=sience),
  Section (!=sound),
  Section (!=tex),
  Section (!=video),
  Section (!=x11),
  Section (!=xfce),
  Section (!=zope)

reprepro_updates: |
  Name: trixie
  Method: https://deb.debian.org/debian
  VerifyRelease: 78DBA3BC47EF2265
  FilterFormula: {{ reprepro_updates_filter_formula }}

  Name: trixie-updates
  Method: https://deb.debian.org/debian
  VerifyRelease: 78DBA3BC47EF2265
  FilterFormula: {{ reprepro_updates_filter_formula }}

  Name: trixie-security
  Method: https://security.debian.org/debian-security
  VerifyRelease: 8E9F831205B4BA95
  FilterFormula: {{ reprepro_updates_filter_formula }}
```

Links
-----

- [reprepro(1)](https://manpages.debian.org/latest/reprepro/reprepro.1.en.html)
- [gpg(1)](https://manpages.debian.org/latest/gpg/gpg.1.en.html)
- [systemd-creds(1)](https://manpages.debian.org/latest/systemd/systemd-creds.1.en.html)
