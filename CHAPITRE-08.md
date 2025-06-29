# CHAPITRE 8 – PROXIES, SÉCURITÉ ET OPTIMISATION DES PERFORMANCES

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant saura :

* Mettre en place un **proxy Zabbix** pour la supervision d’un site distant.
* Sécuriser les communications entre les composants avec **TLS**.
* Configurer les accès utilisateurs de façon sécurisée (RBAC).
* Optimiser les performances de la base et du serveur Zabbix.
* Surveiller et maintenir un Zabbix en production.

---

## 2. Partie théorique

### 2.1. Qu’est-ce qu’un Proxy Zabbix ?

Un **Zabbix Proxy** est un relais intermédiaire qui collecte des données depuis un site distant ou un réseau local, puis les transmet au serveur Zabbix principal.

Utilité :

* Supervision de sites distants sans liaison directe continue
* Réduction de la charge sur le serveur principal
* Meilleure tolérance aux coupures réseau

Modes :

* **Actif** : le proxy envoie les données au serveur
* **Passif** : le serveur vient interroger le proxy

Composants :

* Base de données locale (SQLite ou MariaDB)
* Fichier de configuration spécifique (`zabbix_proxy.conf`)

---

### 2.2. Sécurisation TLS (chiffrement)

Zabbix permet de sécuriser les communications entre :

* Agent ↔ Serveur
* Agent ↔ Proxy
* Proxy ↔ Serveur

Modes :

* PSK (Pre-Shared Key)
* Certificats TLS

Types de certificats :

* Auto-signé
* Fournisseur (Let’s Encrypt, etc.)

---

### 2.3. Gestion des utilisateurs et des droits

* **Groupes d’utilisateurs** : opérateurs, admins, readonly
* **Permissions par hôte/groupe de hosts**
* **Restrictions par interface ou tableau de bord**

Bonne pratique : ne jamais utiliser le compte `Admin` en production pour tous les utilisateurs.

---

### 2.4. Optimisation des performances

Zabbix peut devenir lourd si mal dimensionné. Points clés :

* **Housekeeping** : nettoyage automatique des données anciennes
* **History/trends** : durée de rétention à ajuster selon les besoins
* **Partitionnement des tables** (PostgreSQL ou MySQL)
* **Load sur le serveur Zabbix** : vérifier les workers/process

Commandes utiles :

```bash
zabbix_server -R config_cache_reload
zabbix_server -R housekeeper_execute
```

---

## 3. LAB 8 – DÉPLOIEMENT D’UN PROXY ZABBIX AVEC CHIFFREMENT TLS

---

### 3.1. Objectif

* Déployer un proxy Zabbix sur une machine distante.
* Le relier au serveur principal en mode **actif**.
* Utiliser **TLS PSK** pour chiffrer les communications.
* Vérifier le bon fonctionnement.

---

### 3.2. Étape 1 – Préparer la machine Proxy

Créer une nouvelle VM Linux (Debian/Ubuntu) :

```bash
sudo apt update && sudo apt install zabbix-proxy-mysql sqlite3 -y
```

Utiliser SQLite pour la base locale :

```bash
mkdir -p /var/lib/zabbix
sqlite3 /var/lib/zabbix/zabbix.db < /usr/share/zabbix-sql-scripts/schema.sql
chown -R zabbix:zabbix /var/lib/zabbix
```

---

### 3.3. Étape 2 – Configuration du proxy

Éditer le fichier :

```bash
sudo nano /etc/zabbix/zabbix_proxy.conf
```

Champs importants :

```
Server=IP_DU_SERVEUR
Hostname=proxy-site-dist
DBName=/var/lib/zabbix/zabbix.db
TLSConnect=psk
TLSAccept=psk
TLSPSKFile=/etc/zabbix/psk.key
TLSPSKIdentity=proxy-site-dist
```

Créer le fichier de clé :

```bash
openssl rand -hex 32 > /etc/zabbix/psk.key
chmod 600 /etc/zabbix/psk.key
```

---

### 3.4. Étape 3 – Ajouter le proxy dans Zabbix Web

Aller dans :
Configuration > Hosts > Proxies > Create Proxy

* Proxy name : proxy-site-dist
* Mode : Active
* TLS PSK identity : proxy-site-dist
* TLS PSK : coller le contenu de `/etc/zabbix/psk.key`

---

### 3.5. Étape 4 – Démarrer le service

```bash
sudo systemctl restart zabbix-proxy
sudo systemctl enable zabbix-proxy
```

Sur le serveur, on doit voir le proxy actif dans :
Monitoring > Latest Data > Hosts by proxy

---

### 3.6. Étape 5 – Supervision d’un hôte via proxy

* Créer un nouvel hôte
* Onglet “Agent interface” : IP du serveur distant
* Onglet “Agent proxy” : sélectionner `proxy-site-dist`
* Appliquer un template
* Vérifier que les données remontent

---

### 3.7. Étape 6 – Vérifier la sécurité

Dans :
Configuration > Hosts > Onglet Encryption

* TLS Connect : PSK
* Identity : proxy-site-dist
* PSK : contenu clé

Redémarrer agent si besoin.

---

## 4. Résultat attendu

* Le proxy communique bien avec le serveur via TLS.
* Des hôtes distants supervisés apparaissent dans Zabbix.
* Le serveur principal est délesté des requêtes locales.
* Les communications sont sécurisées avec PSK.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre un proxy actif et un proxy passif ?
2. Quels sont les modes de sécurisation des communications dans Zabbix ?
3. Qu’est-ce que le housekeeping ?
4. Pourquoi utiliser SQLite pour un proxy ?

### Exercice pratique

1. Mettre en place un second proxy passif sur un autre site
2. Configurer une base MariaDB au lieu de SQLite
3. Mesurer la charge CPU du serveur Zabbix avec et sans proxy (items `system.cpu.util`)
