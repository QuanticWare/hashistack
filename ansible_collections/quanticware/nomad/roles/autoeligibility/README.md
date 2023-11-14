README

# Description
Configure autounseal

## How role works (summary)
Ansible will install ansible on target host and install systemd service that runs at boot to unseal Nomad

&nbsp;

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook deploy minimal example

### Playbook install example

```
- name: Vaut configure Auto Auto Eligibility
  hosts: $HOSTS
  become: yes
  tasks:
  - name: Nomad | Install Nomad autounseal
    ansible.builtin.import_role:
      name: quanticware.nomad.autounseal
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | nomad | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |
| fqdn |  "vaut" | your DOMAIN.TLD to access |


### Nomad info

|   var   |   default | note |
|   ---   |   --- | --- |
| nomad\_install\_http\_scheme | "https" | Self-explanatory |
| nomad\_install\_http\_ip | "127.0.0.1" | Self-explanatory |
| nomad\_install\_http\_port | 8200 | Self-explanatory |
| nomad\_install\_dc\_name | "dc1" | Self-explanatory |
| nomad\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| nomad\_install\_tls\_cert |  "\{\{ nomad\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| nomad\_install\_tls\_privatekey |  "\{\{ nomad\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
