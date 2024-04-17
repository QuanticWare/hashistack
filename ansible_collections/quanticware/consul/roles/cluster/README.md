README
# Description
Deploy Consul from Hashicorp: [Consul](https://www.consul.io/)
Only single node currently

## How role works (summary)
Ansible will:
- Generate TLS Certs
- Install Consul
- Create ACL
- Create Token
- Will write on Ansible controller host credentials

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
        name: quanticware.consul.install
```

# Vars

## defaults/main.yml

### General
| var | default | note |
| --- | --- | --- |
| consul\_install\_http\_scheme |  "https" | Self-explanatory |
| consul\_install\_http\_ip |  "127.0.0.1" | Self-explanatory |
| consul\_install\_http\_port |  "8501" | Self-explanatory |
| check\_online\_version |  true | If needed to check on Hashicorp website before install |
| consul\_config\_file |  "/etc/consul/config.json" | Self-explanatory |
| consul\_install\_dc\_name |  "dc1" | Self-explanatory |

### TLS
| var | default | note |
| --- | --- | --- |
| consul\_install\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| consul\_install\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| consul\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| consul\_install\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| consul\_install\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| consul\_install\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| consul\_install\_tls\_cert |  "\{\{ consul\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| consul\_install\_tls\_privatekey |  "\{\{ consul\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
| consul\_install\_tls\_common\_name |  "*.\{\{ consul\_install\_dc\_name \}\}.consul" | Common name for CSR |
| consul\_install\_tls\_subject\_alt\_name |  "DNS | localhost,IP:127.0.0.1,DNS:server.dc1.consul" | Subject Alt Name for CSR |
