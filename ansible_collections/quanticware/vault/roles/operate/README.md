README
# Description
[Vault](https://vaultproject.io/) Manage Secrets & Protect Sensitive Data. Secure, store and tightly control access to tokens, passwords, certificates, encryption keys for protecting secrets and other sensitive data using a UI, CLI, or HTTP API.


## How role works (summary)

Ansible run action define by variable role_action

According to role_action variable:

read: It will read secret and store in variable vault_secret_value

write: Write secret from ansible dict vault_secret variable

unseal: Unseal Vault with keys found in variable: vault_keys_base64 (on ansible controller or on host locals facts)

token_create_nomad: Create a new token and if needed add to Nomad

kv_engine_create: Create Key/Value Engine

# Requirements

## System
* Vault 1.10 +
* Nomad 1.2 +

And with surprise! Ansible 2.10 + with community.general collection

## Ansible
In your ansible environnement, need to set some mandatory vars:
* _vault_root_token_ : root token to interact with Vault

And some almost mandatory:
* _role_action_ : Which action should be run
* _vault_keys_base64_ : Keys uses to unseal Vault

# Playbook deploy minimal example

```
- name: Vault operate write
  hosts: $HOST
  become: yes
  tasks:
    - name: Import Vault Operate
      ansible.builtin.import_role:
        name: quanticware.vault.operate
      vars:
        role_action: write
        vault_secret_path: "apps/data/{{ fqdn }}"
        fqdn: "domain.tld"
        admin: "admin_username"
        password: "very_secret_password"
        vault_secret_to_get: password
        vault_secret:
          data:
            fqdn: "{{ fqdn }}"
            admin: "{{ admin }}"
            password: "{{ password }}"
```

```
- name: Vault operate Read
  hosts: $HOST
  become: yes
  tasks:
    - name: Import Vault Operate
      ansible.builtin.import_role:
        name: quanticware.vault.operate
      vars:
        role_action: read
        vault_secret_path: "apps/data/{{ fqdn }}"
        vault_secret_to_get: password
```

```
- name: Vault operate Create/Renew Token
  hosts: $HOST
  become: yes
  tasks:
    - name: Import Vault Operate
      ansible.builtin.import_role:
        name: quanticware.vault.operate
      vars:
        role_action: token_create
        nomad_token: true
```

```
- name: Vault operate Create KV
  hosts: $HOST
  become: yes
  tasks:
    - name: Import Vault Operate
      ansible.builtin.import_role:
        name: quanticware.vault.operate
        tasks_from: kv_engine_create
      vars:
        vault_kv_name: example
```

NOTE: Nomad token is based on quanticware.hashivault.deploy architecture


```
- name: Vault operate unseal
  hosts: $HOST
  become: yes
  tasks:
    - name: Import Vault Operate
      ansible.builtin.import_role:
        name: quanticware.vault.operate
      vars:
        role_action: unseal
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| vault\_http\_scheme | https | Self-explanatory |
| vault\_http\_ip | 127.0.0.1 | Self-explanatory |
| vault\_http\_port | 8200 | Self-explanatory |

| vault\_operate\_log\_level | INFO | Log level, INFO | DEBUG |
| vault\_operate\_prometheus\_retention\_time | 24h | Self-explanatory |

| check\_online\_version |  true | If needed to check on Hashicorp website before install |

| vault\_operate\_dc\_name |  "dc1" | Self-explanatory |

&nbsp;
### TLS
| var | default | note |
| --- | --- | --- |
| vault\_operate\_tls\_ca\_host | localhost | Which host will use to generate CA authority |
| vault\_operate\_tls\_ca\_host\_dir |  "\{\{ project\_dir \}\}/env/host\_vars/\{\{ inventory\_hostname \}\}/tls" | Where to put CA keys |
| vault\_operate\_tls\_ca\_pubkey |  "hashistack-ca.pem" | Name of CA cerficate |
| vault\_operate\_tls\_ca\_privatekey |  "hashistack-ca-key.pem" | Name of CA private key |
| vault\_operate\_tls\_ca\_provider |  "ownca" | Which type of CA will use to create cert (don't change) |
| vault\_operate\_tls\_host\_certificate\_dir |  "/etc/ssl/hashistack" | Where to put CA pubey on target host |
| vault\_operate\_tls\_cert |  "\{\{ vault\_operate\_dc\_name \}\}-server-consul.pem" | Name of Consul TLS Cert |
| vault\_operate\_tls\_privatekey |  "\{\{ vault\_operate\_dc\_name \}\}-server-consul.key" | Name of Consul TLS Private key |

### Read specs

|   var   |   default | note |
|   ---   |   --- | --- |
| vault\_secret\_path | "apps/data/{{ fqdn }}" | For write and read actions, where is store secrets |
| vault\_secret\_to\_get |  password | Which secret to store in valut vault\_secret_value, secret find in vault\_secret_path |

### Write specs
|   var   |   default | note |
|   ---   |   --- | --- |
| vault\_secret |  - | mandatory to write secret |
| &nbsp;  &nbsp; data  |  - | mandatory to write secret |
| &nbsp;  &nbsp; &nbsp;  &nbsp; variable  |  - | a dict of values to write to secret |

### Token
|   var   |   default | note |
|   ---   |   --- | --- |
| nomad\_token |  True | Does the new token is for Nomad? |
| nomad\_conf\_dir |  "/etc/nomad.d" | If nomad_token == true, where is configuration dir of Nomad |
| token\_role\_name |  nomad | Role uses for new token |
| token\_ttl |  "768h" | Time to leave of new token |
| vault\_policies |  "nomad-server" | List of policies use for new token |


### KV
|   var   |   default | note |
|   ---   |   --- | --- |
| vault\_kv\_name | "" | Name of KV engine to enable |
