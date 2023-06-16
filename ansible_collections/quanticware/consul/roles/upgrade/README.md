README
# Description
Upgrade [Consul](https://consul.io) for an manual installation

Not ready for Consul install from APT Hashicorp repository

## How role works (summary)
Ansible will:
- Check on [Hashicorp Release page](https://releases.hashicorp.com/consul) if `check_online_version` is set to `true`
- Download binary according to host architecture
- Unzip in path `/usr/local/bin` by default
- Restart Consul

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +
* Consul 1.14 + with TLS

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook operate minimal example

```
- name: Upgrade Consul
  hosts: $HOST
  become: yes
  tasks:
    - name: Upgrade Consul
      ansible.builtin.import_role:
        name: quanticware.consul.upgrade
```
