# CHAPITRE 1 – INTRODUCTION À LA SUPERVISION ET À ZABBIX

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Comprendre les fondements de la supervision informatique.
* Expliquer l’architecture technique de Zabbix.
* Installer une plateforme Zabbix fonctionnelle sur un serveur Linux.
* Accéder à l’interface web et effectuer les premiers paramétrages.

---

## 2. Partie théorique

### 2.1. Qu’est-ce que la supervision ?

La supervision est le processus de suivi et de contrôle de l'état d'un système informatique. Elle permet d'anticiper les incidents, de surveiller la performance et de réagir rapidement en cas de panne.

**Deux types principaux de supervision** :

* **Supervision active** : le serveur interroge périodiquement les équipements.
* **Supervision passive** : les agents ou équipements envoient les données au serveur.

**Différences supervision vs monitoring** :

* Supervision = surveillance en temps réel + alertes
* Monitoring = collecte de métriques (souvent pour l’analyse a posteriori)

### 2.2. Pourquoi utiliser Zabbix ?

Zabbix est une solution **open source**, gratuite et extrêmement puissante, qui permet :

* La surveillance de serveurs, applications, bases de données, équipements réseau, cloud, etc.
* La gestion centralisée des alertes.
* L'automatisation de la découverte des équipements.
* Des tableaux de bord graphiques pour l’analyse.

### 2.3. Architecture Zabbix

**Composants principaux** :

* **Zabbix Server** : cœur du système, collecte les données.
* **Zabbix Agent** : installé sur les hôtes à superviser, envoie les métriques.
* **Base de données** : stocke les données collectées (MySQL/MariaDB/PostgreSQL).
* **Frontend Web** : interface d’administration (PHP).
* **Zabbix Proxy** (facultatif) : relais intermédiaire pour sites distants.

### 2.4. Flux de fonctionnement

```
[Agent Zabbix sur hôte] <--> [Zabbix Server] <--> [Base de données]
                                     |
                                  [Frontend Web]
```

---

## 3. LAB 1 – INSTALLATION D’UNE PLATEFORME ZABBIX

### 3.1. Objectif du lab

Installer un serveur Zabbix fonctionnel sur Ubuntu Server (22.04 LTS recommandé), comprenant :

* MariaDB
* PHP
* Apache2
* Zabbix Server + Zabbix Frontend
* Zabbix Agent sur la même machine

---

### 3.2. Pré-requis

* Une machine virtuelle Ubuntu Server (4 Go RAM minimum, 2 vCPU, 20 Go disque).
* Un accès root (`sudo`) ou utilisateur administrateur.
* Connexion à Internet.

---

### 3.3. Étapes détaillées

#### Étape 1 – Mettre à jour le système

```bash
sudo apt update && sudo apt upgrade -y
```

#### Étape 2 – Installer le serveur de base de données (MariaDB)

```bash
sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

#### Étape 3 – Sécuriser MariaDB (optionnel mais conseillé)

```bash
sudo mysql_secure_installation
```

> Choisir un mot de passe root, désactiver l’accès anonyme, supprimer la base test.

#### Étape 4 – Créer la base de données Zabbix

```bash
sudo mysql -uroot -p
```

Puis dans le shell SQL :

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'motdepassefort';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
FLUSH PRIVILEGES;
EXIT;
```

#### Étape 5 – Installer le dépôt Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
sudo apt update
```

#### Étape 6 – Installer les composants Zabbix

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

#### Étape 7 – Importer le schéma de base de données

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

#### Étape 8 – Modifier la configuration du serveur Zabbix

Éditer le fichier :

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Ligne :

```
DBPassword=motdepassefort
```

#### Étape 9 – Démarrer les services

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

---

### 3.4. Étape finale – Configuration Web

1. Ouvrir un navigateur :
   `http://<IP_VM>/zabbix`

2. Suivre les étapes :

   * Vérification des prérequis PHP
   * Paramètres de base de données
   * Configuration du serveur Zabbix
   * Création du fichier `zabbix.conf.php` (automatique si permissions OK)

3. Se connecter :

   * Nom d’utilisateur : `Admin`
   * Mot de passe : `zabbix` (à changer après installation)

---

## 4. Résultat attendu

* Une plateforme Zabbix accessible via navigateur.
* Une base de données fonctionnelle.
* L’agent Zabbix actif sur le serveur local.
* L’interface prête pour l’ajout de nouveaux hôtes.

---

## 5. Évaluation de fin de chapitre

**Questions de révision** :

1. Quelle est la différence entre supervision active et passive ?
2. Que contient le fichier `zabbix_server.conf` ?
3. Pourquoi séparer base de données et serveur Zabbix ?
4. Quels ports sont utilisés par défaut par Zabbix (server, agent, frontend) ?

**Exercice pratique** :

* Installer une VM Debian à part et installer uniquement l’agent Zabbix dessus. Vérifier depuis le serveur si elle répond sur le port 10050.
