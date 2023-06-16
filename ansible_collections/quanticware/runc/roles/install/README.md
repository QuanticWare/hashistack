# Description
Install [runC](https://github.com/opencontainers/runc) from source

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]


# Playbook deploy minimal example

```
- name: runC Install
  hosts: $HOST
  become: yes
  tasks:
    - name: Install runC
      ansible.builtin.import_role:
        name: quanticware.runc.install
```
