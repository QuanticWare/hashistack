README
# Description
Operate on [Consul](https://consul.io) with API

Only single node currently

## How role works (summary)
Ansible will:
- Call API on Consul to register or deregister service

&nbsp;

# Requirements

## System
* Ubuntu 20.04 +
* Consul 1.14 + with TLS


And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook operate minimal example

For create policy

```
- name: Consul Service
  hosts: $HOST
  become: yes
  tasks:
    - name: "Create Consul policy"
      ansible.builtin.import_role:
        name: "quanticware.consul.operate"
        tasks_from: create_policy
      vars:
        consul_token_name: "example-service"
        consul_policy_name: "example-service"
        consul_policy_rules: 'key_prefix \"example\" {\n  policy = \"write\"\n}\n\nservice \"example\" {\n  policy = \"write\"\n}\n\nagent_prefix \"\" {\n  policy = \"read\"\n}\n\nnode_prefix \"\" {\n  policy = \"read\"\n}\n\nservice_prefix \"\" {\n  policy = \"read\"\n}\n'
```

For create token

```
- name: Consul Service
  hosts: $HOST
  become: yes
  tasks:
    - name: "Create Consul token"
      ansible.builtin.import_role:
        name: "quanticware.consul.operate"
        tasks_from: create_token
      vars:
        consul_token_name: "example-service"
        consul_policy_name: "example-service"
```

```
- name: Consul Service
  hosts: $HOST
  become: yes
  tasks:
    - name: Register a Service in Consul
      ansible.builtin.import_role:
        name: quanticware.consul.operate
        tasks_from: service_register
      vars:
        fqdn: $FQDN
        consul_service_address: $IP
        consul_service_port: $PORT
        traefik_access: "onpremise"
```


For Deregister Service

```
- name: Consul Service
  hosts: $HOST
  become: yes
  tasks:
    - name: Deregister a Service in Consul
      ansible.builtin.import_role:
        name: quanticware.consul.operate
        tasks_from: service_deregister
      vars:
        fqdn: $FQDN
```

&nbsp;
# Vars

## defaults/main.yml
&nbsp;
### General
| var | default | note |
| --- | --- | --- |
| consul\_http\_scheme |  "https" | Self-explanatory |
| consul\_http\_ip |  "127.0.0.1" | Self-explanatory |
| consul\_http\_port |  "8501" | Self-explanatory |
| check\_online\_version |  true | If needed to check on Hashicorp website before install |
| consul\_config\_file |  "/etc/consul/config.json" | Self-explanatory |
| consul\_operate\_dc\_name |  "dc1" | Self-explanatory |

&nbsp;
### TLS
| var | default | note |
| --- | --- | --- |
| consul\_operate\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| consul\_operate\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| consul\_operate\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| consul\_operate\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| consul\_operate\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| consul\_operate\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| consul\_operate\_tls\_cert |  "\{\{ consul\_operate\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| consul\_operate\_tls\_privatekey |  "\{\{ consul\_operate\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
| consul\_operate\_tls\_common\_name |  "*.\{\{ consul\_operate\_dc\_name \}\}.consul" | Common name for CSR |
| consul\_operate\_tls\_subject\_alt\_name |  "DNS | localhost,IP:127.0.0.1,DNS:server.dc1.consul" | Subject Alt Name for CSR |

&nbsp;
### Service
| var | default | note |
| --- | --- | --- |
| fqdn | "hostname.com" |  |
| consul\_service\_name | "\{{ fqdn \| replace('.', '-') \}}" |  |
| consul\_service\_port | "12700" |  |
| consul\_service\_address | 172.17.0.1 |  |
| consul\_operate\_enabletagoverride | "false" |  |

&nbsp;
### Policy and Token
| var | default | note |
| --- | --- | --- |
| consul_token_name | "example-service" | Self-explanatory |
| consul_policy_name | "example-service" | Self-explanatory |
| consul_policy_rules | See Example playbook | To create json online format from HCL file, use: `jq -Rsc . rules.hcl > rules.json` |

&nbsp;
### Traefik Tags
| var | default | note |
| --- | --- | --- |
| traefik\_tags | true | Enable tags for Traefik (Work in progress) |
| traefik\_access | "onpremise" | Where traefik will redirect traffic: "cloud" (https) / "onpremise" (LAN access) |
| service\_name | "\{{ fqdn \| replace('.', '-') \}}" | Mandatory to keep replace filter  |
| traefik\_consul\_connect | false | Use consul connect to secure link between nomad services |
| traefik\_noindex | true | Prevent from bot indexer |
| htaccess\_user | "admin" | User for basic auth |
| htaccess\_password | "123456" | Password for basic auth |
| allow\_ip | 207.180.232.70 | Authorized IP to access to service |
| security\_basic_auth | false | Enable basic auth to access to app |
| security\_ip | false | Enable authorized IP to access to app |
| metrics\_exporter | true | Add tag "prometheus.metrics.enable" in consul traefik config to be used by prometheus |
| app\_name | none | Application name to place in tag |
