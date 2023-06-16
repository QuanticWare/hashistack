README

# Description
Configure autounseal

## How role works (summary)
Ansible will install ansible on target host and install systemd service that runs at boot to unseal Vault

&nbsp;

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook deploy minimal example

### Playbook install example

```
- name: Vaut configure Auto Unseal
  hosts: $HOSTS
  become: yes
  tasks:
  - name: Vault | Install Vault autounseal
    ansible.builtin.import_role:
      name: quanticware.vault.autounseal
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | vault | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |
| fqdn |  "vaut" | your DOMAIN.TLD to access |


### Vault info

|   var   |   default | note |
|   ---   |   --- | --- |
| vault\_install\_http\_scheme | "https" | Self-explanatory |
| vault\_install\_http\_ip | "127.0.0.1" | Self-explanatory |
| vault\_install\_http\_port | 8200 | Self-explanatory |
| vault\_install\_dc\_name | "dc1" | Self-explanatory |
| vault\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| vault\_install\_tls\_cert |  "\{\{ vault\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| vault\_install\_tls\_privatekey |  "\{\{ vault\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
