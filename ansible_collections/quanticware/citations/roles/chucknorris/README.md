README
# Description
THE MOST IMPORTANT ROLE YOU NEED TO HAVE!

Collection of humorous quotes

Ideal for finishing a playbook with joy and good humor rather than just having the stats.

collection reserved for formulas of humor and derision. Neither politics, nor religion, nor defamatory remarks.

## How role works (summary)
Ansible will display une debug a random quote.

&nbsp;
# Requirements

## System
* Ubuntu 20.04 +

And with surprise! Ansible [core 2.14.1]

&nbsp;
# Playbook deploy minimal example

For CA:

```
- name: "Hashistack | Congratulations!"
  ansible.builtin.import_role:
    name: quanticware.citations.chucknorris
  vars:
    philosopher: "$NAME_OF_FRIEND"
    language: "fr"
```

# Vars

## defaults/main.yml

### Roles vars

| var | default | note |
| --- | --- | --- |
| app\_name | citations | Self-explanatory |
| role\_user | root | user for ansible specifics tasks |

&nbsp;
### certificate info
| var | default | note |
| --- | --- | --- |
| philosopher | "Chuck Norris" | Display Name in quotes |
| language | "fr" | Which language for quotes (FR / EN)|
