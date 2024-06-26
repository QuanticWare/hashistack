# HASHISTACK Install

Bonjour :wave:

## TL;DR

With minimals `SSH` and `Ansible` Knowledges.

Ansible requiered collections:
* `ansible-galaxy collection install ansible.netcommon`
* `ansible-galaxy collection install ansible.utils`
* `ansible-galaxy collection install community.general`

&nbsp;

For Hashistack Cluster installation, create bridge for internals communications. By default, bridge named `br0` is used for installation.

Refer in Nomad collection, the [Network Tasks](https://git.quanticware.com/quanticware/hashistack/src/branch/main/ansible_collections/quanticware/nomad/roles/install/tasks/04_network.yml) for example.

&nbsp;

```sh
git clone https://git.quanticware.com/quanticware/hashistack.git
```

&nbsp;

* Adapt variables `project_dir` in `env/group_vars/all.yml`
* Update inventory directories for each node
* Update variables in `env/host_vars/$HOSTTARGET/all.yml` for each node
* Update `env/inventory.ini`
* Update `hosts:` according to your inventory in playbooks in playbook directory
* Run `ansible-playbook playbook/hashistack_single_node.yml`
* Or Run `ansible-playbook playbook/hashistack_cluster.yml`

&nbsp;

After installation. Find all news credentials in `env/host_vars/$HOSTTARGET/hashistack.yml` for single node mode or `env/group_vars/$GROUP/hashistack.yml` for cluster.

&nbsp;

Run
```sh
ssh -i ~/.ssh/$SSH_PRIVATE_KEY $USER@$HOSTTARGET -L 4646:127.0.0.1:4646 -L 8501:127.0.0.1:8501 -L 8200:127.0.0.1:8200
```
And open your web browser and go to `127.0.0.1:4646` to reach Nomad Web UI for example.

&nbsp;

It's over


&nbsp;

## What is Hashistack ?

It's a stack of services developed by [HashiCorp](https://www.hashicorp.com) to deploy and manage your softwares (Kubernetes alternative) on Ubuntu Server 20.04 minimal (prefer Ubuntu Server 22.04)

* [Nomad](https://www.nomadproject.io) : workload orchestrator to deploy and manage
* [Consul](https://www.consul.io) : like a network directory to connect apps with service mesh
* [Vault](https://www.vaultproject.io) : secrets manager

&nbsp;

## Skills required

* SSH Knowledge
* Ansible basic install
* Coffee

&nbsp;

## How it work

Ansible will do for you:

* Install [Docker](https://www.docker.com) (enabled by default)
* Install [CNI](https://github.com/containernetworking/cni) for container networking
* Install [Podman](https://podman.io) (disabled by default)
* Create Certificate Authority on your Ansible controller to generate TLS certs
* Install Consul with ACL/TLS support
* Install Vault with ACL/TLS support
* Install Nomad with ACL/TLS support
* Configure firewall to open ports 22,80,443
* Configure firewall to open ports for cluster
* Quote a great philosopher

&nbsp;

## How to Install ?

To install the Hashistack, we need an automation tools, [Ansible](https://www.ansible.com)!

I won't explain here how to install and configure [Ansible](https://www.ansible.com), there is already so many tutorials about this. Little tips, read [the official documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html), use `pip` inside a virtual python environment for more flexibility and security.

&nbsp;

On your target host, configure your unix account with sudo grant access to execute Ansible . I suggest you to use this little bash script to do that (Don't forget to Adapt `$USER` and `$SSH_PUB_KEY`.):

```
#!/bin/bash

useradd -m -s /bin/bash -c "Ansible" -r $USER

mkdir -p /home/$USER/.ssh
echo "$SSH_PUB_KEY" >> /home/$USER/.ssh/authorized_keys
chown -R $USER:$USER /home/$USER/.ssh
chmod -R 600 /home/$USER/.ssh

echo $USER ALL=NOPASSWD: ALL >> /etc/sudoers.d/$USER
chmod 400 /etc/sudoers.d/$USER
```

&nbsp;

Find a place on your computer and git clone this project.

```sh
git clone https://github.com/QuanticWare/hashistack.git
```

Go to `hashistack` directory or open this folder in your [IDE](https://vscodium.com).

&nbsp;

Now you will see a usual ansible tree structure. Read Ansible docs to understand all this structure in detail.

First, you need to configure some variables. Go to `env/group_vars/all.yml` and update it with your local path:

```yml
project_dir: "~/labo/hashistack"
```

&nbsp;

Now, go into `env/host_vars` folder. You will see a folder named `hosttarget` you can rename it as you want, to have more friendly name according to your infrastructure.
Update `$IP`, `$USER` from  `all.yml` as well.
Update `env/inventory.ini`, change `hosttarget`variable.

And for cluster install repeat operation for each node in `env/hots_vars`
Update `env/inventory.ini`, change `dev_cluster`variable.

At this point, 2 ways:

1. Let variable `hashistack_roles_auto_assign == true` and and the role will calculate for you and with the value of variable `hashistack_server_fault_tolerance: 1` ([Read this article for more informations](https://www.hashicorp.com/blog/resilient-infrastructure-with-nomad-fault-tolerance-outage-recovery)) the role distribution. Example: for 6 hosts in group, 3 will be assign as `server` and 3 to `client`.

2. Or if you need a specific roles distribution, continue to read this README.

&nbsp;

And for each node of cluster, you can defined role. Example: `hashistack_node_role: "server"` or `hashistack_node_role: "client"`

Instead of `hashistack_node_role` variable, you can choose for each component, a specific role. Example:
```
consul_node_role: server
nomad_node_role: server
vault_node_role: server
```

or

```
consul_node_role: client
nomad_node_role: client
vault_node_role: none # vault doesn't have 'client' role mode
```

&nbsp;

### Optional:
You can, if you want, disable TLS and/or ACL configuration with these variables (default: true):
```
hashistack_acl: false
hashistack_tls: false
```


But it's **stronly discouraged** for production.

&nbsp;

Now go to the `playbook` folder, you will find a beautiful and complex playbook: `hashistack.yml`, edit it and change:

```yml
  hosts: hosttarget
```

with the name of your target you will have in the `env/inventory.ini` file.

Same operations for cluster.

&nbsp;

From your terminal, go to hashistack folder, (start your engine!) and run:

```sh
ansible-playbook playbook/hashistack_single_mode.yml
```

or

```sh
ansible-playbook playbook/hashistack_cluster.yml
```

Take of coffee and enjoy `Hashistack Install` show in the terminal!

After a few minutes, you will get a good quote of Chuck Norris which means your Hashistack is ready!

Congrats!

But now...

&nbsp;

## How to access to your Hashistack UI components?

Now, the Hashistack is ready, and to access to UI, I purpose this solution: `SSH port forward`.

You can modify you SSH config on your Ansible controller, for example, `vim ~/.ssh/config` and add:

```
 Host $HOSTTARGET
  Hostname $IP
  User $USER
  Port 22
  IdentityFile ~/.ssh/$SSH_PRIVATE_KEY
  PreferredAuthentications publickey
  LocalForward 8501 127.0.0.1:8501
  LocalForward 8200 127.0.0.1:8200
  LocalForward 4646 127.0.0.1:4646
```

Change `$HOSTTARGET`, `$IP`, `$SSH_PRIVATE_KEY` according to your `env/host_vars/$HOSTTARGET/all.yml`

After save and close, in your terminal, enter: `ssh $HOSTTARGET` and open your prefered [Web Browser](https://www.mozilla.org/fr/firefox/new) and go to `https://127.0.0.1:4646`, you will be warned about self signed certificate, accept and reach the Nomad Web UI!

This is another one liner method to do it:

```
ssh -i ~/.ssh/$SSH_PRIVATE_KEY $USER@$HOSTTARGET -L 4646:127.0.0.1:4646 -L 8501:127.0.0.1:8501 -L 8200:127.0.0.1:8200
```

&nbsp;

## How to authentificate against components?
When you are on  UI, for example Nomad, on top right, click on sign in and on next page, you will be asked to provide management key. But, where is this management key? Simply in `env/host_vars/$HOSTTARGET/hashistack.yml` this new file appears during the Hashistack installation. You will find within all credentials you need to interact with Hashistack components.

&nbsp;

## How to use CLI?

In ssh, to use nomad CLI:

```
~# nomad status
Error querying jobs: Unexpected response code: 400 Client sent an HTTP request to an HTTPS server.
```
As you can see, there is issue...

You need to pass somes arguments to cli, but to be simplier, add environment variables, this is only for you! Don't say to anyone!

```
export CONSUL_HTTP_ADDR=127.0.0.1:8501
export CONSUL_HTTP_SSL=true
export CONSUL_HTTP_TOKEN={{ consul_acl_master_token }}
export CONSUL_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export CONSUL_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-consul.pem
export CONSUL_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-consul.key

export NOMAD_ADDR=https://127.0.0.1:4646
export NOMAD_TOKEN={{ nomad_management_token }}
export NOMAD_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export NOMAD_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-nomad.pem
export NOMAD_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-nomad.key

export VAULT_ADDR=https://127.0.0.1:8200
export VAULT_TOKEN={{ vault_root_token }}
export VAULT_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export VAULT_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-vault.pem
export VAULT_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-vault.key
```

In this example, replace sur `{{ xxx }}` variables according to the `env/host_vars/$HOSTTARGET/hashistack.yml`

And now, CLI commands will work as expected! Example:

```
~# consul members
Node               Address         Status  Type    Build   Protocol  DC   Partition  Segment
hashistack-server  127.0.0.1:8301  alive   server  1.15.2  2         dc1  default    <all>
```

&nbsp;

## Tips

### Consul intentions

By default Consul Mesh Services block all communications between services. But in dev or home lab, you can for testing, allow all communications. When you register a job need to use Consul Mesh services, to avoid issue, create intention by Consul UI that allow all services to acces to all services.

### Reverse proxy

This is mandatory to deploy apps. A reverse proxy will a single HTTP(s) entrypoint for your softwares backend. If you want, you can use my [Traefik](https://traefik.io/) Ansible role (coming soon).

### Vault Nomad token renew

The `vault_nomad_token` have a limited Time To Live, 32 days maximun. After that, if you restart or run a new job, you will get **failed** status. You can find in `ansible_collections/quanticware/vault/roles/operate` a playbook to renew token automatically.

### Operate

To interact with all Hashistack API, you will find a custom Ansible role called `operate` in every collections `ansible_collections/quanticware/`. See the `README.md` to understand how they work.

&nbsp;

# THANKS TO

I spent many hours to do that and I would thanks few friends, [Mathieu Garcia](https://github.com/wiseflat) and [Carmelo Ingrao](https://github.com/carmelo42) for helping me doing this awesome stack, [The Hashicorp community](https://discuss.hashicorp.com/) for their time ;-)

I will try to improve this role and add cluster mode in a future version

Thanks you all!


&nbsp;


&nbsp;

---

Dans la langue de Molière
---

&nbsp;


# Installer une HASHISTACK

Bonjour :wave:

## Qu'est-ce qu'une Hashistack ?

Il s'agit d'une suite de logiciels développés par [HashiCorp](https://www.hashicorp.com) qui part leur synergie permet le déploiement et la gestion de micro-services dans des contaires. (Une alternative à Kubernetes) à partir de Ubuntu 20.04, mais de préférence Ubuntu 22.04 LTS.

* [Nomad](https://www.nomadproject.io) : Orchestrateur pour déploiement et de maintien en conditions opérationnelles.
* [Consul](https://www.consul.io) : muti-services comme annuaire de services réseaux, découverte de services, stockage clés/valeurs etc.
* [Vault](https://www.vaultproject.io) : gestionnaire de secrets avancé.

### Avertissement:

Actuellement, l'installation de la Hashistack est limité à un seul hôte, le mode distribué sera développé plus tard, oui, un peu plus tard, durant le siècle actuel promis!

&nbsp;

## Connaissances requises

* Savoir se servir de SSH
* Des connaissances basiques de Ansible
* Du café! (avec ou sans sucre)

&nbsp;

## Que va faire ce rôle?

Pour tout fonctionne Ansible va:

* Installer [Docker](https://www.docker.com) (activé par défaut)
* Installer [CNI](https://github.com/containernetworking/cni) pour le réseau des containers
* Installer [Podman](https://podman.io) (désactivé par défaut)
* Créer une Autorité de Certificat (CA) sur le contrôlleur Ansible pour générer les certificats TLS
* Installer Consul avec le support des ACL/TLS
* Installer Vault avec le support des ACL/TLS
* Installer Nomad avec le support des ACL/TLS
* Configurer le parefeu en ouvrant les ports 22,80,443
* Configurer le parefeu en ouvrant les port pour le mode cluster
* Citer un Grand Philisophe contemporain! Chuck Norris!

&nbsp;

## Comment installe-t-on ?

Pour installer la Hashistack, nous avons besoin d'un outils d'automatisation, et quoi d'autres que [Ansible](https://www.ansible.com)?!

Je ne vais pas vous expliquer ici comment installer et configurer [Ansible](https://www.ansible.com), Il y a déjà beaucoup de tutoriels qui le feront bien mieux. Petite astuce, lisez [la documentation officielle](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html), et utilisez `pip` dans un environement virtuel python pour plus de flexibilité et sécurité.

&nbsp;

Sur l'hôte cible, il faut configurer un compte unix avec des droits sudo pour exécuter Ansible, je vous propose ce petit script bash pour faire cela (N'oubliez pas d'adapter `$USER` et `$SSH_PUB_KEY`.):

```
#!/bin/bash

useradd -m -s /bin/bash -c "Ansible" -r $USER

mkdir -p /home/$USER/.ssh
echo "$SSH_PUB_KEY" >> /home/$USER/.ssh/authorized_keys
chown -R $USER:$USER /home/$USER/.ssh
chmod -R 600 /home/$USER/.ssh

echo $USER ALL=NOPASSWD: ALL >> /etc/sudoers.d/$USER
chmod 400 /etc/sudoers.d/$USER
```

&nbsp;

Maintenant trouvez un endroit qui vous plaît sur votre contrôleur Ansible et git clone le projet.

```sh
git clone https://github.com/QuanticWare/hashistack.git
```

Allez dans le répertoire `hashistack` avec l'explorateur ou ouvrez-le avec votre [IDE](https://vscodium.com) préféré.

&nbsp;

Vous allez y découvrir une aborescence standard d'un projet Ansible. Lisez la doc d'Ansible pour mieux comprendre en détails.

Dans un premier temps, vous allez devoir configurer quelques variables.

Dans `env/group_vars/all.yml` changez cette variable avec le chemin local:

```yml
project_dir: "~/labo/hashistack"
```

&nbsp;

Maintenant dans le répertoire `env/host_vars`. Vous allez voir un répertoire nommé `hosttarget`, vous pouvez le renommer avec un nom qui sera plus en adéquation avec votre infrastructure.

Mettez à jour `$IP`, `$USER`, `Port` dans `all.yml` avec le compte unix SSH configuré sur l'hôte cible.
Mettez à jour dans `env/inventory.ini`, changez `hosttarget`par le nom du répertoire dans `env/host_vars` que vous avez changé précédement.

Même opération pour le mode cluster. En configurant chaque node dans `env/host_vars` .

&nbsp;

Maintenant dans le répertoire `playbook`, vous trouverez un beau et complexe playbook: `hashistack.yml`, éditez le en changeant:

```yml
  hosts: hosttarget
```

avec le nom que vous avez configuré dans `env/inventory.ini`.

Aussi pour le mode cluster en modifiant le nom du group `dev_cluster`.

A cette étape, 2 possibilités:

1. Laisser la variable `hashistack_roles_auto_assign == true` et le role calculera pour vous avec la valeur de la variable `hashistack_server_fault_tolerance: 1` ([Lire cet article pour plus d'infos (EN)](https://www.hashicorp.com/blog/resilient-infrastructure-with-nomad-fault-tolerance-outage-recovery)) l'attribution des rôles. Exemple: Pour 6 hosts dans le groupe, 3 seront assignés comme `server` et 3 comme `client`.

2. Ou si vous avez des besoins spécifiques pour l'assignation des rôles, continuez à lire ce README.

&nbsp;

Pour chaque nodes du cluster, vous devez désigner un rôle. Exemple: `hashistack_node_role: "server"` ou `hashistack_node_role: "client"`

A la place de la variable `hashistack_node_role`, vous pouvez choisir pour chaque composants un rôle. Exemple:
```
consul_node_role: server
nomad_node_role: server
vault_node_role: server
```

ou

```
consul_node_role: client
nomad_node_role: client
vault_node_role: none # vault n'a pas de mode 'client'
```
&nbsp;

### Optional:
Vous pouvez si vous le souhaitez, désactiver la configuration du TLS et/ou des ACL avec ces variables (Par défaut: **true**)
```
hashistack_acl: false
hashistack_tls: false
```


Mais c'est ***fortement déconseillé*** dans une production.

&nbsp;

Ca sent bon! On va pouvoir démarrer le moteur! Dans le terminal, allez dans le répertoire `hashistack` et lancez:

```sh
ansible-playbook playbook/hashistack_single_mode.yml
```

ou

```sh
ansible-playbook playbook/hashistack_cluster.yml
```

Prenez vous un petit café ou une bière (à consommer avec modération) et appréciez le spectacle dans le terminal!

Après quelques minutes, si vous voyez une citation du Grand Chuck Norris, c'est que votre Hashistack est prête au service!

Félicitations!

Mais maintenant?

&nbsp;

## Comment accéder à l'interface des commosants de la Hashistack?

Maintenant que votre Hashistack est prête à l'empoloi, pour accéder à l'interface web, je vous propose cette première solution, le `SSH port forward`.

Vous pouvez modifier sur votre contrôleur Ansible, par exemple `vim ~/.ssh/config` et ajouter:

```yml
 Host $HOSTTARGET
  Hostname $IP
  User $USER
  Port 22
  IdentityFile ~/.ssh/$SSH_PRIVATE_KEY
  PreferredAuthentications publickey
  LocalForward 8501 127.0.0.1:8501
  LocalForward 8200 127.0.0.1:8200
  LocalForward 4646 127.0.0.1:4646
```

Changez `$HOSTTARGET`, `$IP`, `$SSH_PRIVATE_KEY` par rapport à vos modifications dans `env/host_vars/$HOSTTARGET/all.yml`

Sauvegardez, fermez et dans votre terminal, tapez: `ssh $HOSTTARGET`, une fois la connexion SSH établie ouvrez votre navigateur préféré [Web Browser](https://www.mozilla.org/fr/firefox/new) et allez sur `https://127.0.0.1:4646`, vous allez voir un avertissement à propos d'un certificat auto-signé, acceptez pour accéder à l'interface web de Nomad!

Ou part cette commande SSH, qui est équivalente:

```ssh
ssh -i ~/.ssh/$SSH_PRIVATE_KEY $USER@$HOSTTARGET -L 4646:127.0.0.1:4646 -L 8501:127.0.0.1:8501 -L 8200:127.0.0.1:8200
```

&nbsp;

## Comment s'authentifier dans les composants de la Hashistack?

Quand vous êtes sur une interface web, par example Nomad, en haut à droite, cliquez sur `sign in` et sur la page suivante, vous sera demandé une clé, Mais où ma trouve-t-on me direz-vous? Tout simplement dans`env/host_vars/$HOSTTARGET/hashistack.yml` qui est apparu par magie durant l'installation de la Hashistack, vous y trouverez les identifiants nécessaires pour chaque composant de la Hashistack.

&nbsp;

## Comment utiliser la CLI?

Avec SSH, pour utiliser Nomad avec les CLI par exemple:

```
~# nomad status
Error querying jobs: Unexpected response code: 400 Client sent an HTTP request to an HTTPS server.
```
Vous retournera cette erreur.

Pour éviter cela, vous pouvez passer des arguments à Nomad, mais plus simplement des variables d'environnements, alors c'est pour vous, mais ne le répétez à personne:

```
export CONSUL_HTTP_ADDR=127.0.0.1:8501
export CONSUL_HTTP_SSL=true
export CONSUL_HTTP_TOKEN={{ consul_acl_master_token }}
export CONSUL_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export CONSUL_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-consul.pem
export CONSUL_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-consul.key

export NOMAD_ADDR=https://127.0.0.1:4646
export NOMAD_TOKEN={{ nomad_management_token }}
export NOMAD_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export NOMAD_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-nomad.pem
export NOMAD_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-nomad.key

export VAULT_ADDR=https://127.0.0.1:8200
export VAULT_TOKEN={{ vault_root_token }}
export VAULT_CACERT=/etc/ssl/hashistack/hashistack-ca.pem
export VAULT_CLIENT_CERT=/etc/ssl/hashistack/dc1-server-vault.pem
export VAULT_CLIENT_KEY=/etc/ssl/hashistack/dc1-server-vault.key
```

Dans cet exemple, remplacez les variables `{{ xxx }}` par ce qu'il y a dans `env/host_vars/$HOSTTARGET/hashistack.yml`

Et maintenant avec la CLI, ça devrait fonctionner! Exemple:

```
~# consul members
Node               Address         Status  Type    Build   Protocol  DC   Partition  Segment
hashistack-server  127.0.0.1:8301  alive   server  1.15.2  2         dc1  default    <all>
```

&nbsp;

## Astuces

### Consul intentions

Quand vous utilisez Consul Mesh Service pour faire communiquer vos services entre eux, il faut configurer les `Consul Intentions` qui permettent de gérer finement les autorisations de communications entre les services. Par défaut, tout est bloqué, mais dans un environnement de dev ou dans votre "homelab" vous pouvez vous permettre de libérer tous les accès entre services pour les tests, en allant dans le menu `Intentions` et en cliquant sur `create` et en choisissant Allow (all services) to (all services).

### Reverse proxy

C'est presque obligatoire quand vous déployez des applications web. Cela permettra de filtrer et sécuriser l'accès à vos applications par des certificats HTTPS. Je vous conseille Traefik qui est très bien intégré à la Hashistack en se servant de Consul comme annuaire. Si vous réclamez fortement public, je vous publierai prochainement mon role d'installation de Traefik intégré à la Hashistack.

### Vault Nomad token renew

Le token `vault_nomad_token` qui permet à Nomad d'accéder à Vault, a un temps de vie limité 32 jours maximun. Dépassé ce délais, si vous redémarrez un job ou créez un nouveau job, vous aurez le droit un à beau **failed**. Vous pourrez trouvez dans `ansible_collections/quanticware/vault/roles/operate` un playbook pour renouvellement le token Vault de Nomad. Libre à vous de l'automatiser à interval régulier.

### Operations courantes

Pour intéragir avec les API de la Hashistack, vous trouverez un rôle Ansible nommé `operate` dans chaque collection des composants de la Hashistack. Le `README.md` pour comprendre leur fonctionnement.

&nbsp;

# THANKS TO

J'ai passé beaucoup beaucoup de temps à créer ce rôle et j'aimerai remercier mes amis [Mathieu Garcia](https://github.com/wiseflat) and [Carmelo Ingrao](https://github.com/carmelo42) pour leurs aides, leur patience et d'avoir été les cobayes de tous les tests afin de réaliser ce projet à propos de la Super Hashistack! Et un dernier remerciement à la [Communité Hashicorp](https://discuss.hashicorp.com/) pour m'avoir apporté des réponses à mes questions et mes problèmes.


En espérant vous avoir aidé avec ce rôle que j'améliorerai avec le temps et en préparant une version distribué dans une future version.

Merci à tout le monde!
