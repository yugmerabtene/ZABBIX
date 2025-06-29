# CHAPITRE 9 – API REST ZABBIX ET INTÉGRATIONS EXTERNES

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant saura :

* Utiliser l’**API REST JSON-RPC** de Zabbix pour effectuer des opérations à distance.
* Automatiser la **création d’hôtes, d’items et de triggers** via des scripts.
* Intégrer Zabbix avec **Grafana**, **Jenkins**, ou **Prometheus**.
* Générer des tableaux de bord externes et des rapports automatisés.

---

## 2. Partie théorique

### 2.1. Qu’est-ce que l’API Zabbix ?

Zabbix expose une **API JSON-RPC 2.0** permettant d’interagir avec tous ses objets :

* Hosts, items, triggers, templates
* Problems, events, alerts
* Actions, users, media types, maps

L’API est utilisée par :

* Les interfaces web (frontend Zabbix)
* Les intégrateurs et devops (CI/CD, dashboards, scripts)

### 2.2. Avantages de l’API

* **Automatisation** de la supervision de centaines d’hôtes.
* **Export/import** de configurations.
* **Déploiement dynamique** dans des environnements cloud, DevOps.
* **Monitoring-as-code** (infrastructure déclarative).

---

### 2.3. Fonctionnement de l’API

Les appels sont faits en HTTP POST vers :

```
http://<IP_ZABBIX>/zabbix/api_jsonrpc.php
```

Structure JSON de base :

```json
{
  "jsonrpc": "2.0",
  "method": "host.get",
  "params": {},
  "auth": "<token>",
  "id": 1
}
```

Étapes typiques :

1. Authentification (`user.login`) → token
2. Requêtes avec ce token (`host.get`, `item.create`, etc.)

---

### 2.4. Authentification

POST `user.login` :

```json
{
  "jsonrpc": "2.0",
  "method": "user.login",
  "params": {
    "user": "Admin",
    "password": "zabbix"
  },
  "id": 1
}
```

Réponse :

```json
{"result": "ed798e8f6e6978c59744c5cfa3d86e3a"}
```

Ce token sera utilisé pour tous les appels suivants.

---

## 3. LAB 9 – SCRIPT PYTHON D’AJOUT AUTOMATIQUE D’HÔTES

---

### 3.1. Objectif

* Utiliser l’API Zabbix via **Python** (requests).
* Ajouter un nouvel hôte avec IP, groupe, interface, et template.
* Lister les hôtes existants.

---

### 3.2. Préparation

Installer le module Python requis :

```bash
pip install requests
```

---

### 3.3. Script : `add_host.py`

```python
import requests
import json

url = "http://<IP_ZABBIX>/zabbix/api_jsonrpc.php"
headers = {'Content-Type': 'application/json-rpc'}

def call_api(payload):
    r = requests.post(url, headers=headers, data=json.dumps(payload))
    return r.json()['result']

# Étape 1 : login
auth_payload = {
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 1
}
auth_token = call_api(auth_payload)

# Étape 2 : ajout d’hôte
new_host_payload = {
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
        "host": "linux-client-02",
        "interfaces": [{
            "type": 1,
            "main": 1,
            "useip": 1,
            "ip": "192.168.1.30",
            "dns": "",
            "port": "10050"
        }],
        "groups": [{"groupid": "2"}],
        "templates": [{"templateid": "10001"}]
    },
    "auth": auth_token,
    "id": 2
}

result = call_api(new_host_payload)
print("Host added:", result)
```

> Remplacer `groupid` et `templateid` par ceux obtenus avec `hostgroup.get` et `template.get`.

---

### 3.4. Lister tous les hôtes (API `host.get`)

```python
host_list_payload = {
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
        "output": ["hostid", "host"]
    },
    "auth": auth_token,
    "id": 3
}

hosts = call_api(host_list_payload)
for h in hosts:
    print(h["hostid"], h["host"])
```

---

### 3.5. Créer un item via API

Méthode : `item.create`

Exemple :

```json
"params": {
    "name": "CPU Load",
    "key_": "system.cpu.load[percpu,avg1]",
    "hostid": "10105",
    "type": 0,
    "value_type": 0,
    "interfaceid": "1",
    "delay": "30s"
}
```

---

### 3.6. Intégration avec Grafana

* Installer **plugin Zabbix pour Grafana** :

```bash
grafana-cli plugins install alexanderzobnin-zabbix-app
```

* Activer dans Grafana > Configuration > Plugins
* Créer une source de données Zabbix avec :

  * URL : `http://<IP_ZABBIX>/zabbix`
  * Authentification utilisateur Zabbix
* Créer des dashboards dynamiques

---

### 3.7. Intégration avec Jenkins ou DevOps

* Appeler les scripts Python ou shell depuis un **pipeline CI/CD**
* Exemple : déploiement d’un nouveau serveur = création automatique dans Zabbix
* Exporter les configurations JSON/XML de templates

---

## 4. Résultat attendu

* Ajout automatisé d’hôtes dans Zabbix via l’API.
* Accès à la liste des hôtes existants par script.
* Visualisation des métriques dans **Grafana** via source Zabbix.
* Intégration dans un processus DevOps pour un déploiement à grande échelle.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la méthode pour s’authentifier à l’API Zabbix ?
2. Quelle est la différence entre `host.create` et `host.update` ?
3. À quoi sert `interfaceid` dans la création d’un item ?
4. Quel est l’avantage d’intégrer Zabbix à Grafana ?

### Exercice pratique

1. Écrire un script qui supprime un hôte Zabbix donné par nom (`host.delete`).
2. Créer automatiquement 3 hôtes de test à partir d’un fichier CSV.
3. Créer un tableau de bord Grafana avec 3 métriques provenant de Zabbix.
