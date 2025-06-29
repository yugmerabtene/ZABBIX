# CHAPITRE 7 – AUTOMATISATION ET DÉCOUVERTE (DISCOVERY)

---

## 1. Objectifs pédagogiques

À la fin de ce chapitre, l’apprenant saura :

* Créer des **règles de découverte réseau** (IP range, port, service).
* Définir des **actions automatiques** à l’ajout d’un hôte.
* Utiliser la **découverte bas niveau (LLD)** pour détecter des disques, interfaces, processus.
* Mettre en œuvre des modèles dynamiques et auto-configurables.

---

## 2. Partie théorique

### 2.1. Découverte réseau (Network Discovery)

Zabbix permet de scanner un réseau IP (ex. : 192.168.1.0/24) et d’y détecter :

* Des hôtes répondant à un **ping** (ICMP)
* Des ports ouverts (TCP 22, 80, 161…)
* Des services (SNMP, agent Zabbix, HTTP…)

**Objectifs** :

* Automatiser l’ajout d’hôtes
* Appliquer un groupe d’hôtes, un template, et des interfaces

La découverte réseau se configure dans :

```
Configuration > Discovery > Create discovery rule
```

---

### 2.2. Actions liées à la découverte

Une fois un hôte découvert, on peut exécuter des **actions automatiques**, comme :

* Créer un hôte
* L’ajouter à un groupe
* Lui appliquer un ou plusieurs templates
* Exécuter un script

Les actions de découverte sont indépendantes des triggers.

---

### 2.3. Découverte bas niveau (LLD)

La LLD (Low-Level Discovery) permet de découvrir dynamiquement :

* Les **systèmes de fichiers**
* Les **interfaces réseau**
* Les **disques montés**
* Les **capteurs matériels**
* Les **conteneurs Docker**

Chaque objet découvert permet à Zabbix de générer :

* Des **items**
* Des **triggers**
* Des **graphiques**
* Des **alertes personnalisées**

Utilisation :

* Via des règles LLD intégrées aux **templates**
* Via **scripts personnalisés** retournant un JSON LLD

---

### 2.4. Format du JSON LLD

Exemple :

```json
{
  "data": [
    { "{#FSNAME}": "/home" },
    { "{#FSNAME}": "/var" }
  ]
}
```

Les macros `{#XYZ}` sont ensuite injectées dans les noms d’items/triggers.

---

## 3. LAB 7 – DÉTECTION AUTOMATIQUE D’HÔTES ET D’INTERFACES RÉSEAU

---

### 3.1. Objectif

* Créer une **règle de découverte réseau** sur un sous-réseau IP.
* Définir une **action automatique** de création d’hôte.
* Appliquer un **template générique** automatiquement.
* Exploiter une **LLD des interfaces réseau** sur un serveur Linux.

---

### 3.2. Étape 1 – Créer une règle de découverte réseau

Aller dans :
Configuration > Discovery > Create Discovery Rule

* Name : Découverte LAN 192.168.1.0/24
* IP Range : `192.168.1.1-254`
* Checks :

  * Zabbix agent
  * ICMP ping
* Delay : 3600s (1 fois par heure)

Enregistrer.

---

### 3.3. Étape 2 – Créer une action liée à la découverte

Aller dans :
Configuration > Actions > Event source : Discovery > Create Action

* Name : Ajout auto - Linux agent
* Conditions :

  * Discovery check = Zabbix agent
* Operations :

  * Add host
  * Add to group : Linux servers
  * Link to templates : Template OS Linux by Zabbix agent

---

### 3.4. Étape 3 – Forcer un hôte à apparaître

Sur une nouvelle VM :

* Installer l’agent Zabbix
* Configurer `Hostname` : auto-detect
* Activer Zabbix-agent

Elle sera détectée à la prochaine exécution de la règle.

---

### 3.5. Étape 4 – Analyser la LLD des interfaces

Aller dans :
Configuration > Hosts > linux-client-01 > Discovery

On y trouve :

* Discovery rule : Network interface discovery
* Prototypes d’items :

  * ifInOctets\[{#IFNAME}]
  * ifOutOctets\[{#IFNAME}]

Chaque interface détectée aura ses propres items et triggers associés.

---

### 3.6. Étape 5 – Créer sa propre règle LLD (avancé)

Créer un **script shell** qui liste les utilisateurs connectés :

```bash
#!/bin/bash
users=$(who | awk '{print $1}' | sort | uniq)
echo -n '{"data":['
for user in $users; do
  echo -n "{\"{#USERNAME}\":\"$user\"},"
done | sed 's/,$//'
echo ']}'
```

L’item Zabbix doit exécuter ce script et renvoyer du JSON LLD.

---

## 4. Résultat attendu

* Des hôtes sont ajoutés automatiquement via la découverte réseau.
* Un template est appliqué automatiquement.
* Les interfaces réseau sont détectées via LLD.
* Des items et triggers sont générés dynamiquement.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre découverte réseau et LLD ?
2. Que permet une action de découverte ?
3. Que représente `{#IFNAME}` dans un item prototype ?
4. Peut-on découvrir dynamiquement des partitions montées ?

### Exercice pratique

1. Créer une règle de découverte LLD personnalisée pour lister les services actifs (via `systemctl list-units`)
2. Ajouter un trigger si un service détecté tombe à l’état "failed"
3. Créer un template avec cette règle LLD et le tester sur plusieurs serveurs
