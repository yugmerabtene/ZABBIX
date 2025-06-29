# CHAPITRE 2 – SUPERVISION D’HÔTES LINUX ET WINDOWS

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Installer et configurer un **agent Zabbix** sur des hôtes Linux et Windows.
* Ajouter des hôtes dans l’interface Zabbix manuellement ou automatiquement.
* Appliquer des **templates** pour surveiller CPU, RAM, disque, processus…
* Comprendre les concepts de **groupes d’hôtes**, **interfaces**, **macros hôte**.

---

## 2. Partie théorique

### 2.1. Qu’est-ce qu’un agent Zabbix ?

Un **agent Zabbix** est un logiciel léger installé sur un hôte (Linux, Windows…) qui envoie des **métriques système** au serveur Zabbix.

Deux versions :

* `zabbix-agent` (classique, fiable, très utilisé)
* `zabbix-agent2` (plus moderne, supporte plugins dynamiques, API JSON native)

Il est possible de faire de la **surveillance sans agent** (SNMP, SSH…), mais l'agent donne plus de détails système et est plus léger en bande passante.

### 2.2. Communication

* Par défaut, l’agent écoute sur le port **10050/tcp**.
* Le serveur Zabbix interroge activement l’agent.
* On peut aussi configurer l’agent pour qu’il envoie (passivement) les données.

### 2.3. Éléments nécessaires à la supervision

* **Nom d’hôte** : doit correspondre à celui déclaré dans l’agent et dans Zabbix.
* **Adresse IP ou DNS** : définie dans l’interface web de Zabbix.
* **Groupe d’hôtes** : permet d’organiser les machines (ex : Linux Servers).
* **Template** : modèle prêt à l’emploi avec des items, triggers, graphs.

---

## 3. LAB 2 – SUPERVISION DE POSTES LINUX ET WINDOWS

---

### 3.1. Objectif du lab

* Installer **zabbix-agent** sur un poste Linux (Debian/Ubuntu).
* Installer **zabbix-agent2** sur une VM Windows (ou WSL).
* Ajouter ces deux hôtes dans l’interface Zabbix.
* Appliquer un **template standard Linux et Windows**.
* Vérifier la réception des métriques.

---

### 3.2. Supervision d’un hôte Linux

#### Étape 1 – Installer l’agent Zabbix

```bash
sudo apt install zabbix-agent -y
```

#### Étape 2 – Modifier la configuration de l’agent

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Modifications importantes :

```conf
Server=<IP_DU_SERVEUR_ZABBIX>
ServerActive=<IP_DU_SERVEUR_ZABBIX>
Hostname=linux-client-01
```

> Le `Hostname` doit être **identique** à celui que vous saisirez dans l’interface Zabbix.

#### Étape 3 – Redémarrer l’agent

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

#### Étape 4 – Vérifier que le port 10050 est ouvert

```bash
sudo ss -tuln | grep 10050
```

---

### 3.3. Supervision d’un hôte Windows

#### Étape 1 – Télécharger l’agent Zabbix

* Télécharger depuis [https://www.zabbix.com/download\_agents](https://www.zabbix.com/download_agents)
* Choisir Windows > zabbix\_agent2 (binaire + conf)

#### Étape 2 – Extraire dans `C:\Zabbix`

#### Étape 3 – Modifier le fichier `zabbix_agent2.conf`

Exemple :

```
Server=192.168.1.10
ServerActive=192.168.1.10
Hostname=windows-client-01
```

#### Étape 4 – Installer le service

Dans PowerShell (en admin) :

```powershell
cd C:\Zabbix
.\zabbix_agent2.exe --config zabbix_agent2.conf --install
.\zabbix_agent2.exe --start
```

#### Étape 5 – Vérification

Dans un navigateur depuis le serveur :

```
telnet windows-client-01 10050
```

---

### 3.4. Ajouter les hôtes dans Zabbix

#### Étape 1 – Se connecter à Zabbix Web

URL : `http://<IP_DU_SERVEUR>/zabbix`

#### Étape 2 – Aller dans :

Configuration > Hosts > Create host

Saisir les informations :

* Hostname : `linux-client-01` (doit correspondre à l’agent)
* Groups : Linux servers
* Interfaces :

  * Type : Agent
  * IP : IP de la machine
  * Port : 10050

Faire pareil pour Windows, avec `windows-client-01` et groupe : Windows servers

---

### 3.5. Appliquer un template

Dans l’onglet "Templates", lier :

* Pour Linux : `Linux by Zabbix agent`
* Pour Windows : `Windows by Zabbix agent`

---

### 3.6. Vérification

Aller dans :

* Monitoring > Hosts > Linux-client-01
* Vérifier que l’état est "ZBX"
* Aller dans Monitoring > Latest data

On doit voir les données de :

* CPU
* Mémoire
* Disque
* Réseau

---

## 4. Résultat attendu

* Les deux hôtes sont bien ajoutés et ont des métriques remontées.
* Les items du template sont actifs.
* Les agents sont visibles dans la section `Monitoring > Hosts`.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre `Server` et `ServerActive` dans `zabbix_agent.conf` ?
2. Qu’est-ce qu’un template dans Zabbix ?
3. Quel est le rôle d’un groupe d’hôtes ?
4. Comment vérifier que l’agent écoute bien sur le port 10050 ?

### Exercice pratique

1. Modifier le fichier de configuration de l’agent Linux pour ajouter un **item personnalisé** qui vérifie l’usage CPU avec `UserParameter`.
2. Créer un hôte supplémentaire sans agent (HTTP check uniquement).
