<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Ergo

This is an [Ansible](https://www.ansible.com/) role which installs [Ergo](https://ergo.chat/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Ergo is a modern IRCd (IRC server software) written in Go.

See the project's [documentation](https://ergo.chat/about) to learn what Ergo does and why it might be useful to you.

## Prerequisites

To enable persistent message history function you can use a [MySQL](https://www.mysql.com/) compatible database server.

If you are looking for an Ansible role for [MariaDB](https://mariadb.org/), you can check out [this role (ansible-role-mariadb)](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Ergo with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# ergo                                                                 #
#                                                                      #
########################################################################

ergo_enabled: true

########################################################################
#                                                                      #
# /ergo                                                                #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Ergo you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
ergo_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Set the network's name

It is also necessary to specify the name of the network in a human-readable name that identifies your network by adding the following configuration to your `vars.yml` file. Make sure not to include white space or special characters.

```yaml
ergo_config_network_name: YOUR_NETWORK_NAME_HERE
```

### Configuring the valid TLS certificate

By default Ergo allows only SSL/TLS connection, and generates a self-signed TLS certificate for it. For the prodution environment it is recommended to set the valid certificate obtained from certificate authorities.

If Traefik is enabled for the container with `ergo_container_labels_traefik_enabled`, the TLS certificate is automatically retrieved from [Let's Encrypt](https://letsencrypt.org/) using the Traefik's certificates resolver.

If you wish to use your own certificate, you can specify the certificate's path to the variables as below after mounting it onto the container:

```yaml
ergo_config_server_listeners_standard_tls_cert: CERTIFICATE_FILE_HERE

ergo_config_server_listeners_standard_tls_key: PRIVATE_KEY_FILE_HERE
```

To use the self-signed certificate, set `fullchain.pem` to the variable for the certificate file and `privkey.pem` for the private key file, respectively.

### Setting server's password

You can also specify the shared password (the IRC `PASS` command) used to connect to the server:

```yaml
ergo_config_server_password: HASHED_PASSWORD_HERE
```

The hashed password can be generated from the plain one by running `htpasswd -nbB '' PLAIN_PASSWORD_HERE | tr -d ':\n'`.

### Setting an operator's password

To set a password for an IRC operator ("oper", "ircop") to perform administrative actions, you can generate a hashed password in the same way as above and specify it to the variable on your `vars.yml` file:

```yaml
ergo_config_opers_admin_password: OPERS_HASHED_PASSWORD_HERE
```

By default it configures the operator named `admin`, and you will be able to log in with `/OPER admin OPERS_PASSWORD_HERE`.

See [this section](https://github.com/ergochat/ergo/blob/master/docs/MANUAL.md#becoming-an-operator) on the manual for details.

### Allowing / disallowing account registration

By default the role is configured to disable users from registering new accounts. You can allow it by adding the following configuration to your `vars.yml` file:

```yaml
ergo_config_accounts_registration_enabled: true
```

### Enabling / disabling messages history

The Ergo instance does not store message history by default. You can enable the history function by adding the following configuration to your `vars.yml` file:

```yaml
ergo_config_history_enabled: true
```

Note that it stores message history in RAM only, and the history does not persist across server restarts.

By connecting Ergo to the MySQL server it becomes possible for Ergo to store it in the persistent database. To have the Ergo instance use your MySQL-compatible database server as the persistent database, add the following configuration to your `vars.yml` file.

>[!NOTE]
> Before enabling this function, please make sure to understand its consequence from the viewpoint of laws like GDPR.

```yaml
# Enable storing messages in a persistent database for later playback
ergo_config_history_persistent_enabled: true

# Save persistent history to a MySQL database
ergo_config_datastore_mysql_enabled: true

# Specify the database configuration
ergo_database_mysql_hostname: YOUR_MYSQL_SERVER_HOSTNAME_HERE
ergo_database_mysql_port: YOUR_MYSQL_SERVER_PORT_HERE
ergo_database_mysql_username: YOUR_MYSQL_SERVER_USERNAME_HERE
ergo_database_mysql_password: YOUR_MYSQL_SERVER_PASSWORD_HERE
ergo_database_name_history: YOUR_MYSQL_SERVER_DATABASE_NAME_HERE
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `ergo_environment_variables_additional_variables` variable. See [this section](https://github.com/ergochat/ergo/blob/master/docs/MANUAL.md#environment-variables) on the manual for details.

The role is configured to mount [`ircd.yaml`](../templates/ircd.yaml.j2) onto the container. If you wish to mount and use yours instead, add the following configuration to your `vars.yml` file to avoid using the default configuration file:

```yaml
ergo_config_custom_enabled: true
```

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Ergo becomes available at the specified hostname like `ircs://example.com:6697`.

To get started, log in to the server with an IRC client. Since the instance is not secured with a password, you might want to consider to set the shared password with `ergo_config_server_password` to authenticate who can log in to the instance.

The account registration is disabled by default. To let users register their own accounts, you need to enable it by setting `ergo_config_accounts_registration_enabled` to `true`.

Refer to [`MANUAL.md`](https://github.com/ergochat/ergo/blob/stable/docs/MANUAL.md) for the instruction about configuring the server. You might also want to have a look at [`USERGUIDE.md`](https://github.com/ergochat/ergo/blob/stable/docs/USERGUIDE.md#introduction) for general information (what IRC is, how you can use the server with an IRC client, etc.)

## Troubleshooting

FAQs are available at <https://github.com/ergochat/ergo/blob/master/docs/MANUAL.md#frequently-asked-questions>.

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu ergo` (or how you/your playbook named the service, e.g. `mash-ergo`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
ergo_config_logging_level: debug
```
