README
# Description
Deploy Nomad from Hashicorp: [Vault](https://vault.io)

Only single node currently

## How role works (summary)
Ansible will:
- Generate TLS Certs
- Install Vault
- Will write on Ansible controller host credentials
- Unseal Vault
- Create policies
- Create token for Nomad

&nbsp;

# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

## Ansible
Recommanded to use with collection:
- quanticware.hashistack.install

&nbsp;
# Playbook deploy minimal example

```
- name: Installation Consul
  hosts: $HOST
  become: yes
  tasks:
    - name: Install Hashistack
      ansible.builtin.import_role:
        name: quanticware.vault.install
```

&nbsp;
# Vars

## defaults/main.yml
&nbsp;
### General
| var | default | note |
| --- | --- | --- |
| vault\_install\_http\_scheme |  "https" | Self-explanatory |
| vault\_install\_http\_ip |  "127.0.0.1" | Self-explanatory |
| vault\_install\_http\_port |  "8200" | Self-explanatory |
| vault\_install_log_level |  "info" | Self-explanatory |
| vault_install_prometheus_retention_time |  "24h" | Rentention time for prometheus log |
| check\_online\_version |  true | If needed to check on Hashicorp website before install |
| vault\_install\_dc\_name |  "dc1" | Self-explanatory |

&nbsp;
### TLS
| var | default | note |
| --- | --- | --- |
| vault\_install\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| vault\_install\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| vault\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| vault\_install\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| vault\_install\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| vault\_install\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| vault\_install\_tls\_cert |  "\{\{ vault\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| vault\_install\_tls\_privatekey |  "\{\{ vault\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
| vault\_install\_tls\_common\_name |  "*.\{\{ vault\_install\_dc\_name \}\}.consul" | Common name for CSR |
| vault\_install\_tls\_subject\_alt\_name |  "DNS:localhost,IP:127.0.0.1,DNS:server.global.nomad,DNS:server.{{ vault\_install\_dc_name }}.nomad" | Subject Alt Name for CSR , caution: DNS:server.global.nomad is mandatory|

&nbsp;
### Consul
| var | default | note |
| --- | --- | --- |
| vault\_install\_backend\_consul\_http\_scheme | "https" | HTTP scheme of Consul backend |
| vault\_install\_backend\_consul\_http\_ip | "127.0.0.1" | ip of Consul backend |
| vault\_install\_backend\_consul\_http\_port | 8501 | Port of Consul backend |
| vault\_install\_backend\_consul\_tls\_cert | "{{ vault\_install\_dc_name }}-server-consul.pem" | Cert to use in TLS mode |
| vault\_install\_backend\_consul\_tls\_privatekey | "{{ vault\_install\_dc_name }}-server-consul.key" | Key to use in TLS mode |
