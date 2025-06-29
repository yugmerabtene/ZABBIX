# SYLLABUS DE FORMATION – SUPERVISION AVEC ZABBIX

## Objectifs de la formation

* Comprendre les concepts fondamentaux de la supervision.
* Installer, configurer et administrer Zabbix.
* Créer des templates, déclencher des alertes, et automatiser la surveillance.
* Superviser des environnements physiques, virtuels, réseaux, cloud et conteneurs.
* Réaliser un projet complet de supervision en fin de formation.

---

## CHAPITRE 1 – Introduction à la supervision et à Zabbix

Contenu :

* Définitions et objectifs de la supervision.
* Principes : disponibilité, performance, notifications.
* Architecture Zabbix : serveur, base de données, interface web, agent, proxy.
* Présentation des cas d’usage courants.
* Découverte de l’interface Zabbix.

Lab 1 – Installation de Zabbix :

* Déploiement sur une machine virtuelle (Ubuntu ou CentOS).
* Installation de la base de données (MariaDB ou PostgreSQL).
* Installation du serveur et de l’interface web.
* Configuration initiale et accès à l’interface.

---

## CHAPITRE 2 – Supervision d’hôtes Linux et Windows

Contenu :

* Fonctionnement des agents Zabbix (agent et agent2).
* Installation, configuration et test des agents.
* Ajout manuel et découverte automatique d’hôtes.
* Groupes d’hôtes, interfaces et association de templates.

Lab 2 – Supervision d’un poste Linux et d’un poste Windows :

* Installation des agents Zabbix.
* Déclaration manuelle des hôtes.
* Utilisation de la découverte automatique.
* Attribution de templates et visualisation des métriques.

---

## CHAPITRE 3 – Items, Triggers et Alertes

Contenu :

* Types d’items : agent, SNMP, HTTP, externe, script.
* Création de triggers avec expressions.
* Gravité, dépendances et logique conditionnelle.
* Création d’actions et envoi de notifications.
* Configuration des médias (email, webhook, Slack, Telegram).

Lab 3 – Création d’un item personnalisé avec alerte e-mail :

* Surveillance d’un processus applicatif (exemple : nginx).
* Définition d’un trigger avec niveau critique.
* Configuration de l’envoi d’e-mail à un administrateur.

---

## CHAPITRE 4 – Templates, macros et réutilisation

Contenu :

* Création et gestion de templates personnalisés.
* Utilisation de macros à différents niveaux.
* Inheritance (héritage de templates).
* Application à plusieurs hôtes.

Lab 4 – Création d’un template CPU/RAM avec macros :

* Création d’un template réutilisable avec seuils dynamiques.
* Application à plusieurs hôtes pour supervision industrialisée.

---

## CHAPITRE 5 – Graphiques, Dashboards et Cartes réseau

Contenu :

* Création de graphiques simples et combinés.
* Écrans dynamiques et tableaux de bord.
* Cartes réseau interactives (maps).
* Création de widgets personnalisés.

Lab 5 – Création d’un dashboard de supervision réseau :

* Affichage de métriques en temps réel.
* Cartographie des serveurs supervisés avec état en couleur.
* Visualisation pour affichage permanent (mode kiosque).

---

## CHAPITRE 6 – Supervision SNMP, HTTP, IPMI, SSH

Contenu :

* Fonctionnement de SNMP.
* Intégration d’équipements réseau via SNMP.
* Requêtes HTTP, commandes SSH et surveillance IPMI.
* Importation de MIBs.

Lab 6 – Supervision d’un switch via SNMP et d’un site web :

* Intégration d’un commutateur réseau avec SNMP.
* Création d’un item HTTP pour vérifier un site distant.

---

## CHAPITRE 7 – Automatisation et Découverte

Contenu :

* Découverte automatique des hôtes (réseau, IP range).
* Découverte bas niveau (LLD).
* Création automatique d’items, triggers, et actions.
* Groupes dynamiques.

Lab 7 – Détection automatique de VMs avec application de templates :

* Script de découverte réseau.
* Création de règles d’actions automatiques.
* Affectation dynamique de groupes et templates.

---

## CHAPITRE 8 – Proxies, Sécurité et Performances

Contenu :

* Utilisation des proxies (actif, passif).
* Sécurisation des communications (chiffrement TLS).
* Optimisation des performances (Housekeeping, prétraitement).
* Gestion de la base de données et haute disponibilité.

Lab 8 – Supervision d’un site distant via proxy Zabbix :

* Installation d’un proxy sur site distant.
* Configuration du chiffrement TLS.
* Supervision décentralisée.

---

## CHAPITRE 9 – API REST Zabbix et Intégrations externes

Contenu :

* Introduction à l’API Zabbix (authentification, requêtes).
* Scripts d’automatisation (Python, Bash).
* Intégration avec Grafana.
* Webhooks et autres connecteurs (Prometheus, Jenkins).

Lab 9 – Automatisation de la création d’hôtes via l’API :

* Script Python utilisant l’API Zabbix.
* Importation automatisée d’une liste de serveurs.
* Visualisation des métriques dans Grafana.

---

## PROJET FINAL – Supervision complète d’une infrastructure

Objectif :

Mettre en place une supervision complète sur une infrastructure composée de :

* Deux serveurs Linux (application et web)
* Une base de données PostgreSQL
* Un switch SNMP
* Un site distant à superviser par HTTP
* Un composant cloud (AWS, Azure ou GCP)
