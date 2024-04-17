README
# Description
Deploy Hashicorp stack: [Nomad](https://www.nomadproject.io/), [Consul](https://www.consul.io/), [Vault](https://www.vaultproject.io/)
Only single node currently

## How role works (summary)
Ansible will deploy each part of hashistack on host and will create on ansible controller in "env/host_vars/{{ inventory_hostname }}" folder a hashistack.yml with all importants informations like token, basic auth creds etc.

Additionals informations:
- Creation of TLS certs for each component

&nbsp;

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

## Ansible
Collection:

- ansible.netcommon
- quanticware.docker
- quanticware.cni
- quanticware.podman
- quanticware.consul
- quanticware.vault
- quanticware.nomad
- quanticware.delivery
- quanticware.citations

&nbsp;
# Playbook deploy minimal example

```
- name: Installation Hashistack
  hosts: $HOST
  become: yes
  tasks:
    - name: Install Hashistack
      ansible.builtin.import_role:
        name: quanticware.hashistack.install
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | hashistack | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |
| hero\_name | Mamadoo | Just for fun congrats |
| hero\_lang | fr | lang for citation |

&nbsp;
### Hashistack info
| var | default | note |
| --- | --- | --- |
| hashistack\_access | "cloud" | "cloud" or "onpremise" (to specify which type of domain will be set) |
| check\_online\_version | true | Let process find latest version of Consul/Vault/Nomad on Hashicorp's websites |
| consul\_version | 1.12.3 | Disabled by default |
| vault\_version | 1.11.1 | Disabled by default |
| nomad\_version | 1.3.2 | Disabled by default |
| hashistack\_deploy\_dc\_name | dc1 | Name of Datacenter |

&nbsp;
### Nomad info
| var | default | note |
| --- | --- | --- |
| hashistack\_install\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| hashistack\_install\_tls\_ca\_host\_dir | "\{\{ project_dir \}\}/env/host_vars/\{\{ inventory_hostname \}\}/tls" | Where to put CA keys |
| hashistack\_install\_tls\_ca\_pubkey | "hashistack-ca.pem" | Name of CA cerficate |
| hashistack\_install\_tls\_ca\_privatekey | "hashistack-ca-key.pem" | Name of CA private key |
| hashistack\_install\_tls\_host\_certificate\_dir | "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| hashistack\_install\_tls\_common\_name | "hashistack" | Name of common name for CSR Cert |


&nbsp;
### Firewall info
|   var   |   default | note |
|   ---   |   --- | --- |
| firewall |  false | Tasks to open ports by UFW |
| ssh\_port |  22 | SSH default port |
