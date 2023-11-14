README
# Description
Upgrade [vault](https://vaultproject.io)

## How role works (summary)
Ansible will for each component set to `true`:
- Check on [Hashicorp Release page](https://releases.hashicorp.com) if `check_online_version` is set to `true`
- Download binary according to host architecture
- Unzip in path `/usr/local/bin` by default
- Restart component

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +
* Consul 1.14 + with TLS
* Vault 1.14 + with TLS
* Nomad 1.4 + with TLS

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook operate minimal example

### Playbook install example

```
- name: Upgrade hashistack
  hosts: *HOSTS
  become: yes
  collections:
    - quanticware.hashistack
  tasks:
    - name: Upgrade Hashistack
      ansible.builtin.import_role:
        name: quanticware.hashistack.upgrade
      vars:
        consul_upgrade: True
        vault_upgrade: True
        nomad_upgrade: True
```
