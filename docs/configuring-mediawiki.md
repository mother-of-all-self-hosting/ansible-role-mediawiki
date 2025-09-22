<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 - 2024 MASH project contributors
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Gergely Horváth
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara
SPDX-FileCopyrightText: 2024 Philipp Homann

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up MediaWiki

This is an [Ansible](https://www.ansible.com/) role which installs [MediaWiki](https://github.com/mediawiki/mediawiki/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

MediaWiki is a lightweight authentication server that provides an opinionated, simplified LDAP interface for authentication.

See the project's [documentation](https://github.com/mediawiki/mediawiki/blob/main/README.md) to learn what MediaWiki does and why it might be useful to you.

## Prerequisites

To run a MediaWiki instance it is necessary to prepare a database. You can use a [SQLite](https://www.sqlite.org/), [Postgres](https://www.postgresql.org/), or [MySQL](https://www.mysql.com/) compatible database server. By default it is configured to use SQLite.

If you are looking for Ansible roles for a Postgres or MySQL compatible server, you can check out [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) and [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable MediaWiki with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# mediawiki                                                            #
#                                                                      #
########################################################################

mediawiki_enabled: true

########################################################################
#                                                                      #
# /mediawiki                                                           #
#                                                                      #
########################################################################
```

### Select a version tag (optional)

Due to the nature of a wiki as a common knowledge base, this role is configured to track `lts` tag of the Docker image. If you wish to use a newer version, free to specify it by adding the following configuration to your `vars.yml` file:

```yaml
mediawiki_version: VERSION_TAG_HERE
```

The supported tags can be found at [this page](https://hub.docker.com/_/mediawiki#supported-tags-and-respective-dockerfile-links).

>[!NOTE]
> MariaDB 12.0.0+ is not supported by MediaWiki 1.44 due to [this bug](https://phabricator.wikimedia.org/T401570).

### Set the hostname

To enable MediaWiki you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
mediawiki_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting MediaWiki under a subpath (by configuring the `mediawiki_path_prefix` variable) does not seem to be possible due to MediaWiki's technical limitations.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `mediawiki_environment_variables_additional_variables` variable

See [the official documentation](https://github.com/mediawiki/mediawiki/blob/main/mediawiki_config.docker_template.toml) for a complete list of MediaWiki's config options that you could put in `mediawiki_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, MediaWiki becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and log in to the instance with the administrator account. You can create additional users (admin-privileged or not) after that via the web frontend. See [this section](https://github.com/mediawiki/mediawiki/blob/main/README.md#usage) on the documentation for details about usage, including a recommended architecture.

For a command line interface, a third party client [MediaWiki-CLI](https://github.com/Zepmann/mediawiki-cli) is available.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu mediawiki` (or how you/your playbook named the service, e.g. `mash-mediawiki`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
mediawiki_environment_variables_mediawiki_verbose: true
```
