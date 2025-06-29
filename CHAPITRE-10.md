# CHAPITRE 10 – INTÉGRATION DE ZABBIX AVEC GRAFANA

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant saura :

* Installer et configurer **Grafana** en tant que plateforme de visualisation externe.
* Intégrer **Zabbix comme source de données Grafana**.
* Créer des **dashboards personnalisés** avec filtres dynamiques.
* Concevoir des **alertes visuelles** dans Grafana.
* Comparer et agréger des données issues de **plusieurs hôtes ou templates**.

---

## 2. Partie théorique

### 2.1. Pourquoi utiliser Grafana avec Zabbix ?

Grafana est une plateforme open source de visualisation de données orientée **temps réel**. Il est utilisé pour :

* Offrir des **interfaces graphiques modernes** pour la supervision.
* Centraliser **plusieurs sources de données** (Zabbix, Prometheus, InfluxDB…).
* Permettre un accès **granulaire** et ciblé aux données (équipes DevOps, métiers, NOC…).
* Générer des **tableaux de bord dynamiques**, des rapports, et des alertes visuelles.

---

### 2.2. Différences avec les dashboards Zabbix

| Fonction             | Dashboards Zabbix       | Dashboards Grafana              |
| -------------------- | ----------------------- | ------------------------------- |
| Source unique        | Oui (Zabbix uniquement) | Oui (multi-sources)             |
| UX/UI                | Standard                | Moderne, responsive             |
| Panels personnalisés | Basique                 | Riches (Graph, Table, Heatmap…) |
| Filtres dynamiques   | Non                     | Oui (multi-hôte, multi-groupe)  |
| Partage externe      | Limité                  | Possible (iframe, URL, PDF)     |

---

## 3. LAB 10 – INSTALLATION DE GRAFANA ET INTÉGRATION DE ZABBIX

---

### 3.1. Objectif

* Installer Grafana.
* Ajouter Zabbix comme **source de données**.
* Créer un **dashboard personnalisé**.
* Ajouter des **panels dynamiques** avec filtres.
* Générer des alertes visuelles.

---

### 3.2. Étape 1 – Installation de Grafana

#### Sous Ubuntu/Debian :

```bash
sudo apt update
sudo apt install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Grafana sera accessible sur :
`http://<IP>:3000`
Login par défaut : `admin / admin`

---

### 3.3. Étape 2 – Installer le plugin Zabbix

```bash
grafana-cli plugins install alexanderzobnin-zabbix-app
sudo systemctl restart grafana-server
```

Puis activer dans :
**Grafana > Configuration > Plugins > Zabbix > Enable**

---

### 3.4. Étape 3 – Ajouter la source de données Zabbix

Aller dans :
**Grafana > Configuration > Data Sources > Add data source**

Sélectionner : **Zabbix**

Configurer :

* URL : `http://<IP_ZABBIX>/zabbix/api_jsonrpc.php`
* Authentification :

  * Username : `Admin`
  * Password : `zabbix`

Valider.

---

### 3.5. Étape 4 – Créer un dashboard personnalisé

Aller dans :
**Dashboards > New > New Dashboard**

Ajouter un **panel** :

* Query : Zabbix
* Mode : Metrics
* Group : Linux Servers
* Host : linux-client-01
* Application : CPU
* Item : CPU Load (1min)

Visualiser → Titre : CPU Usage

---

### 3.6. Étape 5 – Ajouter un panel avec filtres dynamiques

Dans le tableau de bord :

* Ajouter une **variable** (`$host`)

  * Type : Query
  * Source : Zabbix
  * Query : `group=Linux Servers`
  * Multi-value : yes
* Revenir sur le panel CPU et remplacer `Host` par `$host`

Ainsi, le graphique devient **interactif** avec sélection multi-hôte.

---

### 3.7. Étape 6 – Créer un panel table avec dernière valeur

Ajouter un panel type : **Table**

* Query :

  * Group : Linux Servers
  * Host : `$host`
  * Application : Memory
  * Item : Available memory

Changer Format : Table → Last value

---

### 3.8. Étape 7 – Ajouter une alerte (Grafana 9+)

Dans un **graph panel** :

* Onglet **Alert**
* Ajouter une règle :

  * Condition : Last > 80
  * Thresholds : rouge à partir de 80%
  * Notification : Email / Slack

Créer un **channel de notification** dans :
**Alerting > Contact points**

---

## 4. Résultat attendu

* Un **dashboard dynamique** dans Grafana interrogeant Zabbix.
* Des panels interactifs avec filtres `$host`, `$group`, `$application`.
* Une **table de valeurs en temps réel**.
* Une **alerte visuelle et par e-mail** si une métrique dépasse un seuil.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre un panel "graph" et "stat" ?
2. Quelle est la différence entre une datasource InfluxDB et Zabbix dans Grafana ?
3. Comment ajouter un filtre `$host` dynamique ?
4. Peut-on créer des alertes dans Grafana à partir des métriques Zabbix ?

---

### Exercice pratique

1. Créer un dashboard "Vue serveur Linux" avec :

   * Panel CPU (graph)
   * Panel mémoire (stat)
   * Panel disque (table)
   * Panel carte réseau (gauge)
2. Ajouter un **filtre dynamique de groupe d’hôtes**
3. Générer un **lien d’export public** (URL/iframe) pour affichage en TV interne

---
