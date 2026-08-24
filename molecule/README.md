<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard MediaWiki installation, backed by SQLite.

### `mariadb`

Tests a standard MediaWiki installation with the MariaDB database.

There is deliberately no Postgres scenario: the official MediaWiki container image ships no `pgsql` PHP extension, so `run.php install --dbtype=postgres` cannot connect to a Postgres server at all. See [the documentation](../docs/configuring-mediawiki.md#prerequisites).

## What the scenarios check

MediaWiki without a `LocalSettings.php` answers HTTP 200 on every route — `api.php` included — with a setup page which even names its version, so a running container proves nothing on its own. Each scenario therefore:

1. converges the role, which leaves an uninstalled wiki behind (installing from `converge.yml` would break the idempotence check)
2. records, in `side_effect.yml`, what that uninstalled wiki answers and what its database holds, and then installs the wiki through the role's own `install-cli-mediawiki` tasks and mounts the resulting `LocalSettings.php` the way the documentation prescribes
3. asserts, in `verify.yml`, that the wiki reports the pinned version and the configured settings through `api.php`, that the administrator account the installer created can log in and create a page through the API, that the page reads back attributed to that account, and that the page is in the database this scenario configured — read with that database's own client, from outside MediaWiki

Because the installation happens in the side effect step, `molecule verify` on its own (without a preceding `molecule side-effect`) has nothing to verify.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
