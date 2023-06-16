README
# Description
Upgrade [nomad](https://nomadproject.io)

Not ready for Consul install from APT Hashicorp repository

## How role works (summary)
Ansible will:
- Check on [Hashicorp Release page](https://releases.hashicorp.com/nomad) if `check_online_version` is set to `true`
- Download binary according to host architecture
- Unzip in path `/usr/local/bin` by default
- Restart nomad

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +
* Nomad 1.14 + with TLS

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook operate minimal example

```
- name: Upgrade nomad
  hosts: $HOST
  become: yes
  tasks:
    - name: Upgrade nomad
      ansible.builtin.import_role:
        name: quanticware.nomad.upgrade
```
