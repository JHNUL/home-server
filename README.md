# Init

Create user to box with vagrant user.

```sh
vagrant ssh
sudo apt update # input vagrant default password
su -l
adduser <username>
# ...
usermod -aG sudo <username>
```

## Ansible

https://spacelift.io/blog/ansible-tutorial

Nopea tapa pistää pystyy rooli
`ansible-galaxy init <role>`