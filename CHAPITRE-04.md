# CHAPITRE 4 – TEMPLATES, MACROS ET INDUSTRIALISATION DE LA SUPERVISION

---

## 1. Objectifs pédagogiques

À la fin de ce chapitre, l’apprenant sera capable de :

* Créer et structurer un **template Zabbix personnalisé** avec items, triggers et graphiques.
* Comprendre le principe d’**héritage** de templates et de liaison avec les hôtes.
* Utiliser les **macros** pour rendre les templates dynamiques et réutilisables.
* Appliquer un template à plusieurs machines de manière industrialisée.

---

## 2. Partie théorique

### 2.1. Qu’est-ce qu’un template ?

Un **template** dans Zabbix est un **modèle réutilisable** qui contient :

* Un ensemble d’**items** (métriques)
* Des **triggers** (conditions d’alerte)
* Des **graphiques**
* Des **applications**
* Des **discovery rules**
* Des **web scenarios**

Ce modèle peut être lié à **un ou plusieurs hôtes**, ce qui permet d’appliquer une supervision homogène.

### 2.2. Avantages d’utiliser des templates

* Centralisation des configurations
* Maintenance simplifiée (1 seul changement impacte tous les hôtes liés)
* Réutilisation à grande échelle
* Standardisation des pratiques de supervision

### 2.3. L’héritage de templates

Un template peut **hériter d’un autre** : c’est-à-dire qu’il peut inclure les éléments d’un ou plusieurs templates liés. Cela permet de créer des templates spécialisés sur la base de templates génériques.

Exemple :

```
Template OS Linux (items système)
   ↳ Template Web Server (items Apache, Nginx)
      ↳ Template App PHP
```

### 2.4. Qu’est-ce qu’une macro Zabbix ?

Une **macro** est une variable définie à plusieurs niveaux (global, template, hôte) et utilisée pour rendre les valeurs dynamiques.

Exemples :

* `{$CPU.THRESHOLD}` : valeur de seuil CPU à définir selon le serveur
* `{$APACHE.PORT}` : port Apache utilisé

Types de macros :

* Macros globales (définies dans Administration > General > Macros)
* Macros de template (dans le template)
* Macros d’hôte (écrasent celles du template si identiques)

---

## 3. LAB 4 – CRÉER UN TEMPLATE PERSONNALISÉ AVEC MACROS

---

### 3.1. Objectif du lab

Créer un **template "Supervision CPU custom"** avec :

* Un item : charge CPU moyenne
* Un trigger utilisant une **macro de seuil**
* Un graphique
* Une application
* Lier ce template à plusieurs hôtes avec un seuil personnalisé

---

### 3.2. Étape 1 – Créer un template

Aller dans :
Configuration > Templates > Create Template

Remplir :

* Template name : Template CPU Custom
* Groups : Templates/Custom
* Linked templates : (aucun pour cet exemple)

Enregistrer.

---

### 3.3. Étape 2 – Créer une application

Cliquer sur le template > Applications > Create Application

* Name : CPU Metrics

---

### 3.4. Étape 3 – Ajouter un item

Aller dans Items > Create Item

* Name : CPU Load (1 min average)
* Type : Zabbix agent
* Key : `system.cpu.load[percpu,avg1]`
* Type of information : Numeric (float)
* Update interval : 60s
* Applications : CPU Metrics

---

### 3.5. Étape 4 – Créer une macro dans le template

Aller dans Macros > Add

* Macro : `{$CPU.LOAD.THRESHOLD}`
* Value : `1.5`

Cette valeur sera le seuil de déclenchement par défaut pour tous les hôtes utilisant ce template.

---

### 3.6. Étape 5 – Créer un trigger

Aller dans Triggers > Create Trigger

* Name : Charge CPU trop élevée sur {HOST.NAME}
* Expression :

```zabbix
{Template CPU Custom:system.cpu.load[percpu,avg1].last()}> {$CPU.LOAD.THRESHOLD}
```

* Severity : Warning
* Dependencies : (optionnel)

---

### 3.7. Étape 6 – Créer un graphique

Aller dans Graphs > Create Graph

* Name : Charge CPU (1 min)
* Items : Ajouter `system.cpu.load[percpu,avg1]` (avec couleur)

---

### 3.8. Étape 7 – Lier le template à un hôte

Aller dans :
Configuration > Hosts > linux-client-01 > Templates > Link new template

* Sélectionner : Template CPU Custom

---

### 3.9. Étape 8 – Surcharger la macro pour ce hôte

Toujours dans l’hôte :

* Aller dans l’onglet Macros > Add
* Macro : `{$CPU.LOAD.THRESHOLD}`
* Value : `2.0` (spécifique à cet hôte)

---

### 3.10. Étape 9 – Simulation

Sur l’hôte Linux, pour augmenter la charge :

```bash
yes > /dev/null &
yes > /dev/null &
```

Attendre quelques cycles. Dans Monitoring > Problems, l’alerte doit apparaître.

---

### 3.11. Nettoyage

Arrêter les processus :

```bash
killall yes
```

---

## 4. Résultat attendu

* Un template réutilisable avec macro dynamique.
* Des métriques CPU collectées.
* Un graphique affichable dans un dashboard.
* Un trigger conditionné par une macro configurable par hôte.

---

## 5. Évaluation de fin de chapitre

### Questions théoriques

1. Quelle est la différence entre une macro de template et une macro d’hôte ?
2. Pourquoi utiliser un template au lieu d’ajouter manuellement chaque item à chaque hôte ?
3. Peut-on lier plusieurs templates à un même hôte ?
4. Que se passe-t-il si un hôte et son template utilisent la même macro ?

### Exercice pratique

1. Créer un **template disque** avec un item sur `/home` (clé : `vfs.fs.size[/home,pfree]`)
2. Définir une macro `{$DISK.FREE.THRESHOLD}` et créer un trigger si < macro
3. Lier ce template à deux hôtes avec des valeurs différentes
