# Description
Install [Podman](https://podman.io) from source

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]


# Playbook deploy minimal example

```
- name: Podman Installation
  hosts: $HOST
  become: yes
  tasks:
    - name: Install Podman
      ansible.builtin.import_role:
        name: quanticware.podman.install
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | go | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |

### Podman vars
| var | default | note |
| --- | --- | --- |
| podman\_buildtags | apparmor seccomp | Flags to compil podman executable |
| podman_rootless |  true | Configure Podman as rootless (recommended) |
| podman_user | "podman" | User name to manage podman in rootless usage |

More informations on Builtags: [Podman docs](https://podman.io/getting-started/installation)
