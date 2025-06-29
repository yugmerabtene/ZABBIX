# CHAPITRE 5 – GRAPHIQUES, DASHBOARDS ET CARTES RÉSEAU

---

## 1. Objectifs pédagogiques

À la fin de ce chapitre, l’apprenant sera capable de :

* Créer des **graphes simples** ou combinés pour les métriques collectées.
* Composer un **tableau de bord Zabbix** adapté à une équipe ou un service.
* Mettre en place une **carte réseau interactive** avec indicateurs d’état.
* Afficher dynamiquement l’état d’un parc de machines.

---

## 2. Partie théorique

### 2.1. Graphiques Zabbix

Zabbix permet de visualiser des **données historiques** sous forme de graphes.

Types de graphes :

* **Simple graph** : lié à un item, généré automatiquement.
* **Custom graph** (graph classique) : plusieurs items affichés.
* **Graph prototype** : utilisé avec les règles de découverte.
* **Widget de type graphique** dans un dashboard.

Paramètres d’un graphe :

* Nom
* Unité (%, MB, etc.)
* Échelle automatique ou fixe
* Couleurs, transparence, type de ligne (plein, pointillé, etc.)

---

### 2.2. Dashboards

Un **tableau de bord Zabbix** est une interface personnalisable composée de **widgets** :

* Graphiques
* Problèmes en cours
* Valeurs récentes
* Cartes réseau
* Triggers récents
* Valeurs calculées
* Vue géographique (si activée)

Chaque utilisateur peut avoir **un ou plusieurs dashboards**.

Un dashboard peut être affiché en **mode kiosque** (plein écran, TV murale).

---

### 2.3. Cartes réseau (Maps)

Les **cartes réseau** permettent de représenter l’état des hôtes sur un plan logique ou physique.

Fonctionnalités :

* Représentation des hôtes avec icône personnalisée
* Affichage d’un lien logique avec état (vert/rouge)
* Récupération de l’état de triggers
* Insertion de groupes, images, zones de texte

Utilisation typique :

* Supervision d’un réseau local
* Représentation d’un datacenter
* Visualisation d’une architecture cloud hybride

---

## 3. LAB 5 – DASHBOARD DE SUPERVISION RÉSEAU INTERACTIF

---

### 3.1. Objectif

* Créer un **dashboard** affichant les métriques clés d’un serveur.
* Ajouter des **widgets dynamiques** (CPU, mémoire, réseau).
* Créer une **carte réseau interactive** représentant l’état de plusieurs hôtes.

---

### 3.2. Étape 1 – Créer des graphes pour un hôte

Aller dans :
Monitoring > Hosts > linux-client-01 > Graphs > Create Graph

Nom : Usage CPU et RAM

Items :

* system.cpu.load\[percpu,avg1]
* vm.memory.size\[pavailable]

Couleurs : attribuer une couleur par métrique.

---

### 3.3. Étape 2 – Créer un dashboard

Aller dans :
Monitoring > Dashboards > Create Dashboard

Nom : Supervision Linux Prod

Définir la taille (par exemple : 6x4)

---

### 3.4. Étape 3 – Ajouter des widgets au dashboard

1. **Graphique personnalisé**

   * Type : Graph
   * Sélectionner le graphe CPU/RAM
   * Titre : Charge serveur Linux 01

2. **Derniers événements**

   * Type : Problems
   * Filtrer par groupe : Linux Servers

3. **Valeur en temps réel**

   * Type : Plain text
   * Item : `vfs.fs.size[/,pfree]`
   * Titre : Espace disque libre /

4. **Carte réseau**

   * Type : Map
   * Carte : Réseau interne (créée ci-dessous)

---

### 3.5. Étape 4 – Créer une carte réseau

Aller dans :
Monitoring > Maps > Create Map

Nom : Réseau interne

Taille : 800x600

Ajoutez des éléments :

* Hosts : linux-client-01, windows-client-01
* Définir des icônes (routeur, serveur…)
* Définir les liens entre les hôtes

Paramètres :

* Trigger label : `{HOST.NAME}: {TRIGGER.NAME}`

Utiliser un trigger de type "host unavailable" pour voir l’état.

---

### 3.6. Étape 5 – Lier la carte au dashboard

Retourner dans Monitoring > Dashboards > Modifier dashboard

Ajouter un widget Map > Choisir : Réseau interne

---

### 3.7. Étape 6 – Activer le mode kiosque

Dans le dashboard > Cliquer sur "Mode kiosque" (plein écran)

Utile pour affichage sur TV ou borne d’état réseau.

---

## 4. Résultat attendu

* Un **dashboard personnalisé** opérationnel.
* Une **visualisation graphique** claire de l’état système.
* Une **carte réseau interactive** avec état de connectivité des hôtes.
* Affichage centralisé des problèmes actifs.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre un graphe simple et un graphe personnalisé ?
2. Quels types de widgets peut-on insérer dans un dashboard ?
3. À quoi sert le mode kiosque ?
4. Quels types d’objets peut-on inclure dans une carte réseau Zabbix ?

### Exercice pratique

1. Créer un **dashboard de supervision Windows** avec :

   * Un graphe CPU + RAM
   * Un widget des erreurs systèmes (problems)
   * Un widget textuel indiquant l’espace disque restant
2. Créer une carte réseau incluant un routeur, un switch, 2 serveurs avec état visuel
3. Configurer l’affichage en plein écran (kiosque) sur un poste client
