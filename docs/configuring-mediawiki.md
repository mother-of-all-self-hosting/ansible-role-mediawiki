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

To run a MediaWiki instance it is necessary to prepare a database.  You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

>[!NOTE]
> It is [not recommended](https://www.mediawiki.org/wiki/Compatibility#Database) to use Postgres, as [this page](https://www.mediawiki.org/wiki/Postgres) on the manual describes that the Postgres support is "second-class" and you may run into some bugs.

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

### Set wiki's name

You also need to specify wiki's name by adding the following configuration to your `vars.yml` file:

```yaml
mediawiki_config_sitename: YOUR_WIKI_NAME_HERE
```

### Specify database

It is necessary to select database used by MediaWiki from a MySQL compatible database, Postgres, and SQLite.

To use a MySQL compatible database, add the following configuration to your `vars.yml` file:

```yaml
mediawiki_database_type: mysql
```

Set `postgres` to use Postgres and `sqlite` to use SQLite, respectively. The SQLite database is stored in the directory specified with `mediawiki_database_path`.

For other settings, check variables such as `mediawiki_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Set wiki's logos (recommended)

Though this is not requirement, the placeholder icons should be replaced with yours by adding the following configuration to your `vars.yml` file:

```yaml
mediawiki_config_logos_1x: 1X_LOGO_PATH_HERE
mediawiki_config_logos_icon: ICON_LOGO_PATH_HERE
```

Both file path and URL can be set to logo paths. See [this page](https://www.mediawiki.org/wiki/Manual:$wgLogos) on the manual about configuration for logos as well.

#### Loading external icon files

You might find [this role (ansible-role-aux)](https://github.com/mother-of-all-self-hosting/ansible-role-aux), maintained by MASH team, helpful for loading logo files to the container.

For example, you can upload ones (here: `1x.png` and `icon.png` inside the local `/home/example/` directory) to `{{ mediawiki_config_path }}` and use them as logos by adding the following configuration to your `vars.yml` file:

```yaml
aux_file_definitions_custom:
  - src: "/home/example/1x.png"
    dest: "{{ mediawiki_config_path }}/1x.png"
    mode: "0640"
    owner: "{{ mediawiki_uid }}"
    group: "{{ mediawiki_gid }}"
  - src: "/home/example/icon.png"
    dest: "{{ mediawiki_config_path }}/icon.png"
    mode: "0640"
    owner: "{{ mediawiki_uid }}"
    group: "{{ mediawiki_gid }}"

mediawiki_config_logos_1x: "$wgResourceBasePath/config/1x.png"
mediawiki_config_logos_icon: "$wgResourceBasePath/config/icon.png"
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `mediawiki_environment_variables_additional_variables` variable

See [the official documentation](https://github.com/mediawiki/mediawiki/blob/main/mediawiki_config.docker_template.toml) for a complete list of MediaWiki's config options that you could put in `mediawiki_environment_variables_additional_variables`.

## Installing

Because installing a MediaWiki instance requires to invoke [`run.php install`](https://www.mediawiki.org/wiki/Manual:Install.php), installation process consists of multiple steps as follows:

### (Unmount LocalSettings.php)

If your `vars.yml` file is configured to mount `LocalSettings.php` file from the previous installation, make sure not to mount it yet — Otherwise the installation command below will fail.

### Installing the service

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

### Installing and configuring MediaWiki

After completing it, run the command below to conduct installation and configuration of MediaWiki by invoking `run.php install`:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=install-cli-mediawiki -e admin_username=ADMIN_USERNAME_HERE -e admin_password=ADMIN_PASSWORD_HERE
```

>[!NOTE]
> Make sure to take a note of the username and password as they are not stored as a plain text.

### Updating LocalSettings.php

If you have changed default configurations on `LocalSettings.php` with variables `mediawiki_config_*`, do not forget to run the command below to update the file:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=adjust-config-mediawiki
```

### Loading LocalSettings.php

Finally, add the following configuration to your `vars.yml` file and restart the service to mount `LocalSettings.php` file inside the MediaWiki's container:

```yaml
mediawiki_container_additional_volumes_auto:
  - type: "bind"
    src: "{{ mediawiki_config_path }}/LocalSettings.php"
    dst: "/var/www/html/LocalSettings.php"
    options: "readonly"
```

As it is fine to set any path to the source as long as it points the generated LocalSettings.php file, the destination path should not be changed.

>[!WARNING]
> Once it is loaded, removing the mount stops your MediaWiki instance from working.

## Usage

After running the command for installation, MediaWiki becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and log in to the instance with the administrator account. You can create additional users (admin-privileged or not) after that via the web frontend. See [this section](https://github.com/mediawiki/mediawiki/blob/main/README.md#usage) on the documentation for details about usage, including a recommended architecture.

For a command line interface, a third party client [MediaWiki-CLI](https://github.com/Zepmann/mediawiki-cli) is available.

## Maintenance

### Updating configuration settings

You can update configuration settings on `LocalSettings.php` to the ones specified on your `vars.yml` file by running the command below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=adjust-config-mediawiki
```

>[!WARNING]
> Running the command overwrites changes directly added to `LocalSettings.php` with default values on `defaults/main.yml`.

### Upgrading MediaWiki

**Before upgrading MediaWiki, make sure to have a look at [the release note](https://www.mediawiki.org/wiki/Release_notes) and [compatibility tables](https://www.mediawiki.org/wiki/Compatibility)**, as sometimes the latest dependencies are not supported. For example, MariaDB 12.0.0+ is not supported by MediaWiki 1.44 due to [this bug](https://phabricator.wikimedia.org/T401570).

After confirming that dependencies are fulfilled on your side, you can run the playbook tag as below to upgrade it by running the maintenance script ([`run.php update`](https://www.mediawiki.org/wiki/Manual:Update.php)):

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=update-mediawiki
```

It should be noted that MediaWiki supports upgrades from up to two LTS releases ago (see [this section](https://www.mediawiki.org/wiki/Upgrading#Check_requirements) on the manual). Upgrading from older versions has to be performed step by step, by editing the `mediawiki_version` variable manually.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu mediawiki` (or how you/your playbook named the service, e.g. `mash-mediawiki`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
mediawiki_environment_variables_mediawiki_verbose: true
```
