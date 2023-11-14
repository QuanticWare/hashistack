README
# Description
[Nomad](https://mariadbproject.io/) from Hashicorp - Orchestrator
Only single node currently


## How role works (summary)

Ansible run action to operate with Nomad API HTTP

create_job: Send job to Nomad API HTTP after convert HCL job file to json

To be use only with `quanticware.deploy` role

# Requirements

## System
* Nomad 1.5+
* JQ
And with surprise! Ansible 2.10 + with

# Playbook deploy minimal example

```
- name: "Deploy {{ app_name }} | Submit Job"
  ansible.builtin.import_role:
    name: quanticware.mariadb.operate
    tasks_from: create_job
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| mariadb\_http\_scheme | https | Self-explanatory |
| mariadb\_http\_ip | 127.0.0.1 | Self-explanatory |
| mariadb\_http\_port | 4646 | Self-explanatory |
| mariadb\_job\_files\_dir | "/var/tmp" | Where are store HCL job on target |

| mariadb\_operate\_log\_level | INFO | Log level, INFO | DEBUG |
| mariadb\_operate\_prometheus\_retention\_time | 24h | Self-explanatory |

| check\_online\_version |  true | If needed to check on Hashicorp website before install |

| mariadb\_operate\_dc\_name |  "dc1" | Self-explanatory |

&nbsp;
### TLS
| var | default | note |
| --- | --- | --- |
| mariadb\_install\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| mariadb\_install\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| mariadb\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| mariadb\_install\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| mariadb\_install\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| mariadb\_install\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| mariadb\_install\_tls\_cert |  "\{\{ mariadb\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| mariadb\_install\_tls\_privatekey |  "\{\{ mariadb\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
