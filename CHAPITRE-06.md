# CHAPITRE 6 – SUPERVISION SNMP, SSH, HTTP, IPMI ET WEB SCENARIOS

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant saura :

* Mettre en place la supervision d’équipements via **SNMP** (switch, routeur, imprimante…).
* Surveiller un serveur distant avec **SSH** sans agent.
* Vérifier la disponibilité d’un site web avec **items HTTP**.
* Définir des **scénarios web** avec étapes et authentification.
* Superviser du matériel via **IPMI** (serveurs physiques).

---

## 2. Partie théorique

### 2.1. Supervision SNMP

**SNMP (Simple Network Management Protocol)** est un protocole permettant à un serveur de collecter des informations auprès d’équipements réseau (switch, routeur, etc.).

* SNMPv1, v2c (communauté : public)
* SNMPv3 (chiffré, authentifié)
* Utilise le port UDP 161

Chaque donnée est identifiée par un **OID** (Object Identifier), exemple :

```
.1.3.6.1.2.1.1.3.0 → Uptime de l’équipement
```

**MIB (Management Information Base)** : fichier de traduction des OID.

---

### 2.2. Supervision SSH (ou commandes distantes)

* Zabbix permet de définir un item de type **"SSH agent"** pour exécuter une commande distante.
* L’hôte cible doit autoriser la connexion SSH depuis le serveur Zabbix.
* L’utilisateur Zabbix doit disposer d’un accès sans mot de passe (clé publique).

---

### 2.3. Supervision HTTP (ou API JSON)

Zabbix peut envoyer des requêtes HTTP GET ou POST pour vérifier :

* La disponibilité d’un site (code 200)
* Le contenu d’une page (regex)
* Les résultats JSON (avec preprocessing)

Cela permet aussi de superviser des **API REST externes**.

---

### 2.4. Web Scenarios

Les **web scenarios** permettent de créer une série d'étapes de test HTTP :

* Exécuter plusieurs requêtes dans l’ordre
* Vérifier la présence d’un mot ou d’un code retour
* Gérer des sessions (authentification)

---

### 2.5. Supervision IPMI

IPMI (Intelligent Platform Management Interface) est utilisé pour interroger des **serveurs physiques** à bas niveau (température, tension, état matériel) sans OS.

* Nécessite des identifiants IPMI
* Type d’item : IPMI agent
* IPMItools doit être installée

---

## 3. LAB 6 – SUPERVISER UN SWITCH EN SNMP ET UN SITE EN HTTP

---

### 3.1. Objectif

* Superviser un switch réseau local avec **SNMPv2** (sans agent).
* Superviser un site web distant ([http://example.com](http://example.com)) via **requête HTTP**.
* Créer un **web scenario** simulant une connexion utilisateur (authentification + page interne).

---

### 3.2. Étape 1 – Activer SNMP sur le switch

Cela dépend du modèle. Exemple Cisco :

```
conf t
snmp-server community public RO
```

IP : 192.168.1.50
Communauté : public

Vérification depuis le serveur Zabbix :

```bash
snmpwalk -v2c -c public 192.168.1.50
```

---

### 3.3. Étape 2 – Ajouter le switch dans Zabbix

Dans l’interface Zabbix :

* Configuration > Hosts > Create host
* Hostname : switch-snmp-01
* Interface : SNMP, IP : 192.168.1.50, port 161
* Group : Réseau

---

### 3.4. Étape 3 – Appliquer un template SNMP

* Lier le template : `Template Net Generic SNMPv2`
* Adapter la communauté SNMP dans Macros :

  * Macro : `{$SNMP_COMMUNITY}` = public

---

### 3.5. Étape 4 – Ajouter un item HTTP

Aller dans :
Configuration > Hosts > linux-client-01 > Items > Create Item

* Name : Disponibilité site example.com
* Type : Simple check
* Key : `web.page.get[example.com]`
* Type of information : Text
* Update interval : 60s

Vérifier dans Monitoring > Latest Data

---

### 3.6. Étape 5 – Créer un web scenario

Configuration > Hosts > linux-client-01 > Web > Create Scenario

* Name : Test site web interne
* Steps :

  * Step 1 : page de login

    * URL : [https://site.test/login](https://site.test/login)
    * Method : POST
    * Variables : username=admin\&password=1234
  * Step 2 : page interne

    * URL : [https://site.test/dashboard](https://site.test/dashboard)
    * Required string : "Bienvenue"

Configurer les options :

* Update interval : 120s
* Retries : 1

---

## 4. Résultat attendu

* Les métriques SNMP du switch sont visibles (bande passante, uptime, etc.).
* Le site [http://example.com](http://example.com) est interrogé automatiquement toutes les minutes.
* Le scénario web renvoie un succès si l’authentification réussit et que la page interne s’affiche.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre un item SNMPv2 et un item SNMPv3 ?
2. Que permet un scénario web par rapport à un simple `web.page.get[]` ?
3. Pourquoi utiliser la supervision HTTP pour des API REST ?
4. Peut-on superviser un serveur sans agent ni SNMP ?

### Exercice pratique

1. Ajouter un item HTTP vérifiant que le JSON retourné par une API contient la clé "status":"ok"
2. Superviser un onduleur ou imprimante réseau via SNMP
3. Créer un item SSH exécutant `uptime` sur un serveur distant et traitant le résultat
