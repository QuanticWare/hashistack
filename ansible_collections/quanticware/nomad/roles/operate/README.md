README
# Description
[Nomad](https://nomadproject.io/) from Hashicorp - Orchestrator
Only single node currently


## How role works (summary)

Ansible run action to operate with Nomad API HTTP

job_create: Send job to Nomad API HTTP after convert HCL job file to json

To be use only with `quanticware.deploy` role

# Requirements

## System
* Nomad 1.5+
* JQ
And with surprise! Ansible 2.10 + with

# Playbook deploy minimal example

To run job:

```
- name: "Deploy {{ app_name }} | Submit Job"
  ansible.builtin.import_role:
    name: quanticware.nomad.operate
    tasks_from: job_create
```

To find alloc ID
```
- name: "Find {{ app_name }} Alloc ID"
  ansible.builtin.import_role:
    name: quanticware.nomad.operate
    tasks_from: find_allocation_id
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| nomad\_http\_scheme | https | Self-explanatory |
| nomad\_http\_ip | 127.0.0.1 | Self-explanatory |
| nomad\_http\_port | 4646 | Self-explanatory |
| nomad\_job\_files\_dir | "/var/tmp" | Where are store HCL job on target |

| nomad\_operate\_log\_level | INFO | Log level, INFO | DEBUG |
| nomad\_operate\_prometheus\_retention\_time | 24h | Self-explanatory |

| check\_online\_version |  true | If needed to check on Hashicorp website before install |

| nomad\_operate\_dc\_name |  "dc1" | Self-explanatory |

&nbsp;
### TLS
| var | default | note |
| --- | --- | --- |
| nomad\_install\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| nomad\_install\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| nomad\_install\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| nomad\_install\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| nomad\_install\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| nomad\_install\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| nomad\_install\_tls\_cert |  "\{\{ nomad\_install\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| nomad\_install\_tls\_privatekey |  "\{\{ nomad\_install\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
