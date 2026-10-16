<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Passbolt

This is an [Ansible](https://www.ansible.com/) role which installs [Passbolt](https://www.passbolt.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Passbolt is an open source password manager for teams, built around OpenPGP.

See the project's [documentation](https://www.passbolt.com/docs/) to learn what Passbolt does and why it might be useful to you.

## Prerequisites

To run a Passbolt instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server or [Postgres](https://www.postgresql.org/).

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Passbolt with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# passbolt                                                             #
#                                                                      #
########################################################################

passbolt_enabled: true

########################################################################
#                                                                      #
# /passbolt                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Passbolt you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
passbolt_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Passbolt under a subpath (by configuring the `passbolt_path_prefix` variable) does not seem to be possible due to Passbolt's technical limitations.

### Configuring database

#### Specify database

It is also necessary to select database used by Passbolt from a MySQL compatible database and Postgres.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
passbolt_database_type: postgres
```

Set `mysql` to use a MySQL compatible database.

For other settings, check variables such as `passbolt_database_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Configuring connection to the database server (optional)

By default the role is configured to establish the connection to the database server via a Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the MySQL compatible server's Unix socket path on the host (bind-mount source)
passbolt_database_mysql_socket_path_host: ""

# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
passbolt_database_postgres_socket_path_host: ""
```

Setting it enables to connect to the database server via Unix socket mounted in the container.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the MySQL compatible server via a Unix socket
passbolt_database_mysql_socket_enabled: false

# Disable the connection to the Postgres server via a Unix socket
passbolt_database_postgres_socket_enabled: false
```

### Configuring the mailer (optional)

Passbolt sends the links to set up an account by email. To configure a SMTP mailer, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
# Specify SMTP server hostname
passbolt_environment_variables_email_transport_default_host: ""

# Specify SMTP server port number
passbolt_environment_variables_email_transport_default_port: 587

# Specify SMTP server username
passbolt_environment_variables_email_transport_default_username: ""

# Specify SMTP server password
passbolt_environment_variables_email_transport_default_password: ""

# Specify the email address that emails will be sent from
passbolt_environment_variables_email_default_from: ""

# Set to `true` to enable STARTTLS
# To connect with implicit TLS (port 465) instead, prefix the hostname with `ssl://` (e.g. `ssl://smtp.example.com`).
passbolt_environment_variables_email_transport_default_tls: true
```

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `passbolt_environment_variables_additional_variables` variable

Refer to [the official documentation](https://www.passbolt.com/docs/hosting/environment-reference/) for a complete list of Passbolt's config options that you can put in `passbolt_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Passbolt becomes available at the specified hostname like `https://example.com`.

To get started, create the first administrator by running the command below on the server (replace the values with your own):

```sh
docker exec passbolt bin/cake passbolt register_user -u EMAIL_ADDRESS_HERE -f FIRST_NAME_HERE -l LAST_NAME_HERE -r admin
```

Replace `passbolt` with the container name if you changed it (e.g. `mash-passbolt` with the MASH playbook). The command prints a link to finish setting up the account, which needs the [Passbolt browser extension](https://www.passbolt.com/download). The same link is sent by email if the mailer is configured.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu passbolt` (or how you/your playbook named the service, e.g. `mash-passbolt`).
