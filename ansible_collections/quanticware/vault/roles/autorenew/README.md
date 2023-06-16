README
# Description
Vault Auto-Renew in a Nomad job.

&nbsp;
## How role works (summary)

Ansible:
* Submit job to Nomad

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +
* Docker 20.10 +
* CNI plugin 1.1.1 + [Github](https://github.com/containernetworking/plugins)
* UFW builtin OS
* Consul 12 +
* Vault 1.10 +
* Nomad 1.2 +

And with surprise! Ansible 2.10 + with community.general collection

&nbsp;
## Ansible
In your ansible environnement, need to set some mandatory vars:
* _ressources_ : Path to \_ressources where nomad, traefik templates are stored

&nbsp;
## Python
Some python dependencies like:
* _python-nomad_ : To deploy nomad job without terraform
* _passlib_ : To generate basic auth for traefik with blowfish encryption

&nbsp;
# Playbook deploy minimal example

```
- name: Deploy Vault Auto-Renew
  hosts: $HOST
  become: yes
  tasks:
    - name: Renew Token Job
      ansible.builtin.import_role:
        name: quanticware.vault.autorenew
      vars:
        vault_token: "$VAULT_TOKEN_TO_RENEW"
```

&nbsp;
# CAVEAT
By default, job nomad will run every 5 of each month. Configure cron if needed according to TTL!

&nbsp;
# Vars

## defaults/main.yml

&nbsp;
### Roles vars

| var | default | note | requiered in user playbook |
| --- | --- | --- | --- |
| app\_name | rundeck | Self-explanatory |  |
| role\_user | root | user for ansible specifics tasks |  |
| install\_method | nomad | Can be: docker, binary etc. be sure to create tasks file in tasks directory with same name.|  |
| fqdn |  "{{ app\_name }}" | your DOMAIN.TLD to access |  |
| http_scheme | http | http or https |  |

&nbsp;
### Ansible vars

| var | default | note | requiered in user playbook |
| --- | --- | --- | --- |
| deploy_credentials | false | if set to True, deploy tasks will include credentials.yml in flow |  |
| deploy_pretasks | false | if set to True, deploy tasks will include pretasks.yml in flow just before submit job | _yes_  |
| deploy_posttasks | false | if set to True, deploy tasks will include pretasks.yml in flow just before submit job |  |


&nbsp;
### Vault Auto-Renew specs

| var | default | note | requiered in user playbook |
| --- | --- | --- | --- |
| vault\_http\_scheme | https | Self-explanatory |
| vault\_http\_ip | 172.17.0.1 | Docker bridge ip to call Vault |
| vault\_http\_port | 8200 | Self-explanatory |
| vault\_operate\_dc\_name |  "dc1" | Self-explanatory |
| vault\_operate\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| vault\_operate\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| vault\_operate\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| vault\_operate\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| vault\_operate\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| vault\_operate\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| vault\_operate\_tls\_cert |  "\{\{ vault\_operate\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| vault\_operate\_tls\_privatekey |  "\{\{ vault\_operate\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |
| vault\_token\_renew\_ttl |  "768h" | By how much to increment the new token Time To Live  |
| vault\_token\_renew\_self |  "true" | If token can self-renew or not, if not, `vault_root_token` is mandatory |


&nbsp;
### Prometheus info
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_metrics\_exporter |  True | Add tag "prometheus.metrics.enable" in consul traefik config to be used by prometheus |

&nbsp;
### Nomad info

More informations: [Nomad Job Docs](https://developer.hashicorp.com/nomad/docs/job-specification/job) and [Nomad Tasks Docs](https://developer.hashicorp.com/nomad/docs/job-specification/task)

|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_group\_name |  "{{ app\_name }}" | Name for group\_01 in group definition |
| group\_01\_nomad\_count |  1 | How many instances nomad have to create |
| group\_01\_image\_tag |  "latest" | Image tag to find in registry
| group\_01\_task\_name | "{{ app\_name }}" | Name for group\_01 in task definition |
| group\_01\_task\_custom |  false | If app need to have custom config at task level, be sure to create group\_01\_task\_custom.yml |
| group\_01\_task\_config\_custom |  false | If app need to have custom config at docker config in task level, be sure to create group\_01\_task\_config\_custom.yml |
| group\_01\_service\_custom |  false | If app need to have additional service , be sure to create group\_01\_service\_custom.yml |
| group\_01\_service\_main\_custom |  false | If app need to have additional settings in main service (where traefik is generally) , be sure to create group\_01\_service\_main\_custom.yml |


|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| job\_name |  "{{ fqdn }}" | Be sure to be uniq for each job in nomad |
| nomad\_url |  "" | Nomad API URL/IP to send job |
| nomad\_http\_auth |  true | If basic auth is actived ([Terraform module](https://registry.terraform.io/providers/hashicorp/nomad/latest/docs)) |
| nomad\_job\_specs\_datacenter |  "dc1" | Self-explanatory |
| nomad\_job\_specs\_region |  "global" | Self-explanatory |
| nomad\_job\_constraint\_node\_unique\_name |  False | To force deploy all groups on only one node|
| nomad\_periodic\_state |  true | If job is periodic like backup job|
| nomad\_periodic |
| &nbsp;  &nbsp;  cron |  "0 0 5 * *" | Self-explanatory, [crontab generator](https://crontab-generator.org/) |
| &nbsp;  &nbsp;  prohibit\_overlap |  "true" | Specifies if this job should wait until previous instances of this job have completed. |
| &nbsp;  &nbsp;  timezone |  "Europe/Paris" | Self-explanatory |


&nbsp;
### Consul info

More informations: [Consul Docs](https://developer.hashicorp.com/consul/docs/connect)

|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_consul\_connect | true | Does Consul will use connect function |
| group\_01\_app\_name |  "{{ app\_name }}" | To add name for group\_01 in tag (can be used by prometheus or others)


&nbsp;
### App specs
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_consul\_connect\_server\_state |  false | Does this group will provide consul connect server to receive requests |
| group\_01\_consul\_connect\_server |
| &nbsp;  &nbsp;  sidecar\_service | - { name: "none", port: 0000 } | name and port, name must be uniq |
| group\_01\_consul\_server\_sidecar\_task\_cpu |  32 | Size in MB |
| group\_01\_consul\_server\_sidecar\_task\_memory |  32 | Size in MB |
| group\_01\_consul\_connect\_client\_state |  false | Does this group will provide consul connect client bind with consul connect server |
| group\_01\_consul\_connect\_client |
| &nbsp;  &nbsp;  sidecar\_service | - { name: "none", port: 0000 } | name and port, name must be uniq |
| group\_01\_consul\_client\_sidecar\_task\_cpu |  128 | Size in MB |
| group\_01\_consul\_client\_sidecar\_task\_memory |  128 | Size in MB |
| consul\_operation |  true | If need to insert info in consul KV |
| consul\_host |  172.26.64.1 | Consul API IP |
| consul\_host\_mesh |  172.26.64.1 | Consul API IP |

&nbsp;
### Vault info
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| vault\_state |  none | Interaction type with vault: read / write / both / none | |
| vault |
| &nbsp;  &nbsp;  policies |  access-creds | Which vault policy to use |
| &nbsp;  &nbsp;  secret |  "apps/data/{{ fqdn }} |
| vault\_path\_mariadb: | "apps/data/mariadb/users" | Path to mariadb users credentials | |
| vault\_secret: | _none_ | |
| &nbsp;  &nbsp; data: | _none_ | Dict of secret to write in Vault |

&nbsp;
### Notification from Ansible

|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| notification | false | Enable or not notification at tasks end |

&nbsp;
### by Mail
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| mail_subject | "Ansible: {{ fqdn }}" | Self-explanatory |
| mail_msg | '\<h3>\<a href="{{ fqdn }}">{{ fqdn }}\</a>\</h3>' | Body message write in HTML |
| mail_to | "{{ operator }}" | Receipient of mail |

&nbsp;
### by RocketChat

|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| rocketchat_msg | "Ansible Tasks Deploy" | Self-explanatory |
| rocketchat_attachments | true | To have frame under |
| rocketchat_attachments_text | "Success!" | First words of frame |
| rocketchat_attachments_title | "Info" | Title of deploy frame |
| rocketchat_attachments_collapsed | "false" | To see all frame without click |
| rocketchat_attachments_fields | _none_ | Dict |
| &nbsp;  &nbsp; | - { short: "true", title: "URL", value: "{{ fqdn }}" } | Content of frame message |

NOTE: This is example to have a prettier notification in RocketChat

&nbsp;
### by NTFY
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| ntfy_msg | \| |  |
| &nbsp;  &nbsp; URL: {{ fqdn }} | |
| &nbsp;  &nbsp; Auth Token: {{ mosquitto_admin_username }} | |
| &nbsp;  &nbsp; Password: {{ mosquitto_admin_password }} | |

&nbsp;
### Firewall info
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| firewall |  false | Tasks to open ports by UFW |

&nbsp;
# vars/main.yml

&nbsp;
### Nomad info|
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| nomad\_job\_update |  "{% if nomad_periodic_state == true %}false{% else %}true{% endif %}" | Does Nomad manages update job |
| nomad\_job\_specs\_type | "{% if nomad_periodic_state == true %}batch{% else %}service{% endif %}" | type of job: service, system, batch, and sysbatch. [Nomad docs](https://developer.hashicorp.com/nomad/docs/schedulers) |

&nbsp;
### Group 01 Nomad general specs
|   var   |   default | note |
|   ---   |   --- | --- |
| group\_01\_nomad\_count | 1 | How many instances nomad will runs|

&nbsp;
### Group 01 Nomad restart specs
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_nomad\_restart\_policy |  True|  Does Nomad manages restart |
| group\_01\_nomad\_restart\_attempts |  10|  Self-explanatory |
| group\_01\_nomad\_restart\_interval |  "5m"|  Self-explanatory |
| group\_01\_nomad\_restart\_delay |  "10s"|  Self-explanatory |
| group\_01\_nomad\_restart\_mode |  "delay"| [Nomad Docs](https://developer.hashicorp.com/nomad/docs/job-specification/restart#mode) |

More informations: [Nomad Docs](https://developer.hashicorp.com/nomad/docs/job-specification/restart)

&nbsp;
### Group 01 Nomad Task
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_task\_driver |  "docker" | Task driver, "docker" only currently.|
| group\_01\_task\_size\_cpu |  256 |  Size in MB |
| group\_01\_task\_size\_memory |  256 |  Size in MB  |

More informations:  [Nomad Docs](https://developer.hashicorp.com/nomad/docs/drivers)

&nbsp;
### Group 01 Nomad network
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_nomad\_install\_webinterface\_port | 80 | Port to access to app (when group\_01\_traefik\_consul\_connect == false) |
| group\_01\_nomad\_network\_state | false |   |
| group\_01\_nomad\_network |

&nbsp;
### Docker

More informations: [Nomad Docs](https://developer.hashicorp.com/nomad/docs/job-specification/volume\_mount)

|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| docker\_compose\_version |  3.9 |  Self-explanatory |
| docker\_gateway |  172.17.0.1|  Self-explanatory |
| docker\_host\_path\_volume | - | Root path on host to create volume |
| docker\_private\_registry\_state |  true |  Does Nomad pull image from private registry|
| docker\_private\_registry\_creds | | |
| &nbsp;  &nbsp;  url |  "xxx"| URL/IP registry |
| &nbsp;  &nbsp;  username |  "xxx"| Self-explanatory  |
| &nbsp;  &nbsp;  password |  "xxx"| Self-explanatory |
| group\_01\_image |  url | Fully qualified name of image |
| group\_01\_volumes\_state |  true| Enable volume mount for group 01 |
| group\_01\_volumes |   - { type: "bind", target: "/etc/ssl/hashistack", source: "/etc/ssl/hashistack", readonly: "false", propagation: "rshared" } | List of mounted volume |


&nbsp;
### Traefik info
|   var   |   default | note | requiered in user playbook |
|   ---   |   --- | --- | --- |
| group\_01\_service\_name |  "{{ fqdn \| replace('-', '') \| replace('.', '') }}"| Mandatory to keep replace filter |
| group\_01\_traefik\_access |  "none" | Where traefik will redirect traffic: "cloud" (https) / "onpremise" (LAN access) / "none" to disable Traefik |
| group\_01\_traefik\_proxy\_port | 80 | When group\_01\_traefik\_consul\_connect == true, which port to access to app |
| group\_01\_traefik\_noindex |  true | Prevent from bot indexer |
| group_01_traefik_tag_service | "{{ fqdn | replace('.', '-') }}-group-01" | Service name in Traefik tag (often fqdn) | |
| group_01_traefik_custom_tags | false | If needed, add custom tags |  |
| group_01_traefik_custom_tags_list |
| &nbsp;  &nbsp;  - '"traefik.http.routers.{{ group_01_service_name }}.service=..."'
| group\_01\_traefik\_consul\_connect |  true | Use consul connect to secure link between nomad services|
| group\_01\_traefik\_sidecar\_task\_cpu |  32 | Self-explanatory |
| group\_01\_traefik\_sidecar\_task\_memory |  32 | Self-explanatory |
| group\_01\_security\_basic\_auth |  False | Enable basic auth to access to app |
| group\_01\_security\_ip | False | Enable authorized IP to access to app|
