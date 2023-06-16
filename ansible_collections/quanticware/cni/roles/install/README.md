README
# Description
Install [CNI](https://github.com/containernetworking/cni) (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.

## How role works (summary)
Ansible will check the CPU architecture and install CNI accordingly.

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook deploy minimal example

```
- name: Install CNI
  hosts: $HOST
  become: yes
  tasks:
  - name: "Install CNI"
    ansible.builtin.import_role:
      name: quanticware.cni.install
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | certificate | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |
