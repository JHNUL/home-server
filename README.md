# Home server setup

Expects Debian 12 (bookworm) as operating system.

Configures and installs all OS and user packages and services in an automated way. Vagrant is used in development mode to setup the infrastructure in a virtual machine.

## Ansible

All these commands must be run in the ansible folder. First activate python virtual environment and install dependencies from requirements.txt.

Setup shell aliases for commands:
```sh
alias abdev='ansible-playbook -i inventory-dev.yaml'
alias abprod='ansible-playbook -i inventory.yaml'
```

### Operating system

#### Copy sources.list file to remote host
```sh
abdev --tags "update_system_packages,sources_list" playbooks/platform.yaml --diff
```

#### Update system packages
```sh
abdev --tags "update_system_packages,apt_update" playbooks/platform.yaml --diff
```

#### Create groups and users
```sh
abdev --tags "create_groups_and_users" playbooks/platform.yaml --diff
```

### Mosquitto broker

#### Install broker
```sh
abdev --tags "install_mosquitto" playbooks/mosquitto.yaml --diff
```

#### Configure broker configuration file
```sh
abdev --tags "configure_mosquitto,mosquitto_conf" playbooks/mosquitto.yaml --diff
```

#### Configure Mosquitto logrotation
```sh
abdev --tags "configure_mosquitto,logrotate" playbooks/mosquitto.yaml --diff
```

#### Configure logfile permissions
```sh
abdev --tags "configure_mosquitto,log_permissions" playbooks/mosquitto.yaml --diff
```

#### Configure Mosquitto systemd service file
```sh
abdev --tags "configure_mosquitto,service_file" playbooks/mosquitto.yaml --diff
```

### Manual checking in the remote box

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
