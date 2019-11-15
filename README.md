# Light Hosting Resilio

An opinionated Resilio setup using Ansible.

## Requirements

- Ansible
- At least one server with ample storage space, preferably multiple

## Setup

### Ansible

Copy `/ansible/group_vars/all/vars.yml.example` as `/ansible/group_vars/all/vars.yml` to configure ansible.  
Copy `/ansible/inventory.example` as `/ansible/inventory` to configure host inventory, both IPs and DNS entries may be used.

The `root_password` and `user_password` should be the password part of a .htaccess user. After the configuration the user `deploy` should be used.

## Quick start

Create configuration files:

```bash
cp ansible/group_vars/all/vars.yml.example ansible/group_vars/all/vars.yml
cp ansible/inventory.example ansible/inventory
```

Fill in configurations in the two files.

### First run against a new server

The first time running the playbook against a new server, a password for the provider-provisioned root user is required.

```bash
cd ansible
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory -e ansible_user=root --ask-pass --limit=resilio01.storage.example.com provision.yml
```

This is only required on the first run, as the password will be changed afterwards to match the one defined in the `vars.yml` file, and SSH keys will be set up.

### Subsequent runs

After the first run, the playbook may be invoked normally, as the proper users are in place.

```bash
cd ansible
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory provision.yml
```

## Adding Resilio shares

To add shares, add the keys in the `vars.yml` file, and run the playbook to apply the configuration on the server.

```yaml
resilio_shares:
  - group: default
    shares:
    - RESILIO_KEY_1
    - RESILIO_KEY_2
  - group: john_smith
    shares:
    - RESILIO_KEY_3
    - RESILIO_KEY_4
```

The groups act as a directory seperator on the server, allowing to group the shares together on the disk, for easy cleanup or grouping them by person if multiple people share servers for increased availability and speed.

The shares will be located on the server under `/home/rslsync/Resilio Sync/${group}/${share-key}`.

It is recommended to only use encrypted keys on the servers, so cheap VPS providers can be used for large storage capacities, treating the servers as untrusted with personal data.

## Inspirations

Light Hosting Resilio is the second iteration of my personal Resilio setup.

The previous iteration used Ansible as well, but required manual setup of shares using the web UI.

After rewriting my Kubernetes hosting setup resulting in [light-hosting-kube](https://github.com/SorenA/light-hosting-kube), I wanted to recreate the Resilio setup using the same resources and add in managed Resilio Folders to minimize the manual setup, and allow management of multiple mirrors with ease.

The roles `fail2ban`, `harden-linux` and `users` were copied from the `light-hosting-kube` setup.

Harden Linux Ansible role was inspired by [Githubixx's ansible-role-harden-linux](https://github.com/githubixx/ansible-role-harden-linux).
