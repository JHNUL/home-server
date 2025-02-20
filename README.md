- [Home server setup](#home-server-setup)
  - [Ansible](#ansible)
    - [Operating system](#operating-system)
      - [Copy sources.list file to remote host](#copy-sourceslist-file-to-remote-host)
      - [Update system packages](#update-system-packages)
      - [Create groups and users](#create-groups-and-users)
    - [Mosquitto broker](#mosquitto-broker)
      - [Install broker](#install-broker)
      - [Configure configuration directory permissions](#configure-configuration-directory-permissions)
      - [Configure broker configuration file](#configure-broker-configuration-file)
      - [Configure Mosquitto logrotation](#configure-mosquitto-logrotation)
      - [Configure Mosquitto systemd service file](#configure-mosquitto-systemd-service-file)
      - [Configure Mosquitto certificates for TLS](#configure-mosquitto-certificates-for-tls)
      - [Configure username and password authentication](#configure-username-and-password-authentication)
    - [Manual checks in the remote box](#manual-checks-in-the-remote-box)
    - [Connect to MQTT Broker](#connect-to-mqtt-broker)


# Home server setup

Expects Debian 12 (bookworm) as operating system.

Configures and installs all OS and user packages and services in an automated way. Vagrant is used in development mode to setup the infrastructure in a virtual machine.

## Ansible

All these commands must be run in the ansible folder. First activate python virtual environment and install dependencies from requirements.txt.

The commands are appended with the `--check` option to prevent accidental copy-paste-run changes.

Setup shell aliases for commands:
```sh
alias abdev='ansible-playbook -i inventory-dev.yaml'
alias abprod='ansible-playbook -i inventory.yaml'
```

### Operating system

#### Copy sources.list file to remote host
```sh
abdev --tags "update_system_packages,sources_list" playbooks/platform.yaml --diff --check
```

#### Update system packages
```sh
abdev --tags "update_system_packages,apt_update" playbooks/platform.yaml --diff --check
```

#### Create groups and users
```sh
abdev --tags "create_groups_and_users" playbooks/platform.yaml --diff --check
```

### Mosquitto broker

In this setup the broker supports encrypted traffic via TLS. So first a CA certificate and key need to be generated. Add the appropriate duration.

```sh
openssl req -new -x509 -days <duration> -extensions v3_ca -keyout ca.key -out ca.crt
```

Generate a private key to be used by the MQTT broker.

```sh
openssl genrsa -out server.key 4096
```

Generate a signing request to send to the CA

```sh
openssl req -out server.csr -key server.key -new
```

Sign the signing request with the earlier created CA

```sh
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days <duration>
```

Following these steps there should be the following files `ca.crt, server.crt, server.key` which need to be placed in the `mosquitto/files` folder.

#### Install broker
```sh
abdev --tags "install_mosquitto" playbooks/mosquitto.yaml --diff --check
```

#### Configure configuration directory permissions
```sh
abdev --tags "configure_mosquitto,permissions" playbooks/mosquitto.yaml --diff --check
```

#### Configure broker configuration file
```sh
abdev --tags "configure_mosquitto,mosquitto_conf" playbooks/mosquitto.yaml --diff --check
```

#### Configure Mosquitto logrotation
```sh
abdev --tags "configure_mosquitto,logrotate" playbooks/mosquitto.yaml --diff --check
```

#### Configure Mosquitto systemd service file
```sh
abdev --tags "configure_mosquitto,service_file" playbooks/mosquitto.yaml --diff --check
```

#### Configure Mosquitto certificates for TLS
```sh
abdev --tags "configure_mosquitto,mosquitto_tls" playbooks/mosquitto.yaml --diff --check
```

#### Configure username and password authentication
```sh
abdev --tags "configure_mosquitto,mosquitto_client_passwords" playbooks/mosquitto.yaml --diff --check
```

When using username and password authentication, the passwordfile can be created with mosquitto_passwd utility.

### Manual checks in the remote box

Check that the service is running with the correct properties
`systemctl show mosquitto.service`

To check an individual property, e.g. User
`systemctl show mosquitto.service --property=User`

Check logs with journalctl
`journalctl -u mosquitto.service`

Search the log file
`egrep -i "searchsomething" /var/log/mosquitto/mosquitto.log`

Tail the log stream
`tail --follow -n 40 /var/log/mosquitto/mosquitto.log`


### Connect to MQTT Broker

Using MQTT Explorer:
 - point to localhost:8883 (Port-forwarding configured in Vagrantfile)
 - enable TLS
 - use the username and password created in pwfile (for now uses username, password authentication)

![kuva](/docs/images/mqtt_explorer.png)