# CHAPITRE 3 – ITEMS, TRIGGERS ET ALERTES

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Créer des **items personnalisés** pour collecter des métriques.
* Concevoir des **triggers** pour détecter des événements critiques.
* Configurer des **actions** conditionnelles pour envoyer des alertes.
* Utiliser les **media types** pour notifier les équipes par e-mail.

---

## 2. Partie théorique

### 2.1. Qu’est-ce qu’un Item ?

Un **item** est une unité de collecte de données. Il définit ce que Zabbix doit surveiller et comment.

**Composants d’un item** :

* `Key` (clé) : exemple `system.cpu.load[percpu,avg1]`
* `Type` : Zabbix agent, SNMP, HTTP, calculé, externe…
* `Interval` : fréquence de collecte (ex. : 60s)
* `History` : durée de stockage des données brutes
* `Trends` : durée de conservation des tendances agrégées

**Types d’items courants** :

* Agent local (Linux/Windows)
* Agent actif
* Item calculé (formule de plusieurs items)
* Script externe (`UserParameter`)

---

### 2.2. Qu’est-ce qu’un Trigger ?

Un **trigger** est une expression logique qui déclenche une alerte en fonction de la valeur d’un ou plusieurs items.

**Exemple** :

```
{linux-client-01:system.cpu.load[percpu,avg1].last()} > 2
```

**Fonctions disponibles** :

* `last()`, `avg()`, `min()`, `max()`, `count()`, `nodata()`

**Gravités** :

* Not classified, Information, Warning, Average, High, Disaster

**Dépendances** :
Permettent d’éviter les alertes en cascade (ex. : ne pas déclencher l’alerte "Apache down" si "Serveur down" est déjà actif).

---

### 2.3. Qu’est-ce qu’une Action ?

Une **action** est une réponse déclenchée automatiquement par un trigger.

**Composants** :

* Conditions (nom de trigger, gravité, groupe d’hôtes…)
* Opérations (envoi d’e-mail, exécution de script…)
* Répétition, escalade

---

### 2.4. Qu’est-ce qu’un Media type ?

Un **media type** est un canal de notification :

* Email (SMTP)
* SMS (via passerelle)
* Telegram, Slack, Webhook
* Exécution de script local

---

## 3. LAB 3 – CRÉER UN ITEM PERSONNALISÉ ET UNE ALERTE PAR E-MAIL

---

### 3.1. Objectif

* Créer un item personnalisé sur un hôte Linux.
* Créer un trigger qui se déclenche si un fichier log contient une erreur.
* Configurer une action qui envoie une alerte par e-mail.

---

### 3.2. Préparation

Assurons-nous que :

* Le serveur Zabbix est opérationnel.
* Un hôte Linux est supervisé avec un agent fonctionnel.
* Le service mail (SMTP local ou Gmail relay) est configuré sur le serveur Zabbix.

---

### 3.3. Étape 1 – Ajouter un item personnalisé (log)

#### Sur l’hôte Linux :

Créer un fichier `/var/log/demo.log`

```bash
echo "OK - system stable" > /var/log/demo.log
```

#### Modifier `zabbix_agentd.conf` :

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Ajouter à la fin :

```
UserParameter=check.log.error,grep -i 'error' /var/log/demo.log | wc -l
```

Redémarrer l’agent :

```bash
sudo systemctl restart zabbix-agent
```

#### Sur l’interface Zabbix :

Aller dans :
Configuration > Hosts > linux-client-01 > Items > Create Item

* Name : Erreurs dans /var/log/demo.log
* Type : Zabbix agent
* Key : `check.log.error`
* Type of information : Numeric (unsigned)
* Update interval : 60s

Enregistrer.

---

### 3.4. Étape 2 – Créer le trigger associé

Aller dans :
Configuration > Hosts > Triggers > Create Trigger

* Name : Trop d’erreurs détectées dans le log
* Expression :

```
{linux-client-01:check.log.error.last()}>0
```

* Severity : High

Enregistrer.

---

### 3.5. Étape 3 – Simuler une erreur

Sur l’hôte Linux :

```bash
echo "ERROR - something failed" >> /var/log/demo.log
```

Dans l’interface Zabbix :

* Allez dans Monitoring > Problems
* L’alerte doit apparaître si le trigger se déclenche

---

### 3.6. Étape 4 – Configurer un Media type e-mail

Aller dans :
Administration > Media types > Email

Configurer :

* SMTP Server : votre serveur SMTP
* SMTP email : [zabbix@votre-domaine.com](mailto:zabbix@votre-domaine.com)

Ajouter un utilisateur avec une adresse email et lui affecter ce media type :

* Administration > Users > Admin > Media

---

### 3.7. Étape 5 – Créer une action

Aller dans :
Configuration > Actions > Event source : Triggers > Create Action

* Name : Alerte erreurs log
* Condition : Trigger severity = High
* Operation : Send to user `Admin` via media type Email

---

### 3.8. Étape 6 – Réinitialiser le problème

Supprimer la ligne "ERROR..." du log :

```bash
sudo sed -i '/ERROR/d' /var/log/demo.log
```

Attendre 1 cycle (60s), le problème doit disparaître de la console.

---

## 4. Résultat attendu

* L’item `check.log.error` collecte des données.
* Le trigger se déclenche lorsqu’une ligne contenant "error" est détectée.
* Une alerte est envoyée par e-mail à l’administrateur.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre un item "agent" et un item "calculé" ?
2. Quelle fonction trigger permet de détecter l’absence de données ?
3. Qu’est-ce qu’une dépendance entre triggers ?
4. Peut-on exécuter un script shell en réponse à une alerte ?

### Exercice pratique

1. Créer un nouvel item `UserParameter` qui mesure l’espace disque restant sur `/home`.
2. Déclencher une alerte si l’espace est inférieur à 1 Go.
3. Créer une action qui affiche un message dans `/tmp/alerte.log` à chaque alerte (script local).
