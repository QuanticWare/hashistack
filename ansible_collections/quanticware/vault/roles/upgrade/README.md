README
# Description
Upgrade [vault](https://vaultproject.io)

Not ready for Consul install from APT Hashicorp repository

## How role works (summary)
Ansible will:
- Check on [Hashicorp Release page](https://releases.hashicorp.com/vault) if `check_online_version` is set to `true`
- Download binary according to host architecture
- Unzip in path `/usr/local/bin` by default
- Restart vault
- Run ansible playbook to unseal

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +
* Vault 1.14 + with TLS

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook operate minimal example

```
- name: Upgrade vault
  hosts: $HOST
  become: yes
  tasks:
    - name: Upgrade vault
      ansible.builtin.import_role:
        name: quanticware.vault.upgrade
```
