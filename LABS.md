### **LAB 1 – Installation de Zabbix**

**Objectif** : Installer une plateforme Zabbix complète sur une VM Ubuntu.

**Étapes** :

1. Mettre à jour la VM Ubuntu.
2. Installer MariaDB, Apache2, PHP.
3. Créer la base `zabbix`, l’utilisateur et importer le schéma.
4. Installer les paquets `zabbix-server`, `frontend`, `agent`.
5. Démarrer les services et accéder à l’interface via `http://<ip>/zabbix`.

---

### **LAB 2 – Supervision d’hôtes Linux et Windows**

**Objectif** : Installer les agents et superviser des machines réelles.

**Étapes** :

1. Installer `zabbix-agent` sur une VM Linux.
2. Installer `zabbix-agent2` sur une machine Windows.
3. Modifier `Hostname` et IP dans les fichiers de config.
4. Ajouter les hôtes dans l’interface web.
5. Appliquer les templates `Linux by Zabbix agent` et `Windows by Zabbix agent`.

---

### **LAB 3 – Items personnalisés et alertes**

**Objectif** : Créer un item basé sur un log et une alerte e-mail si une erreur est détectée.

**Étapes** :

1. Créer un fichier `/var/log/demo.log`.
2. Ajouter un `UserParameter` dans le fichier de conf agent.
3. Définir un item `check.log.error` dans Zabbix.
4. Créer un trigger si `> 0`.
5. Configurer un `Media Type` (SMTP) et une `Action` pour envoyer un e-mail.

---

### **LAB 4 – Template avec macros dynamiques**

**Objectif** : Créer un template CPU avec seuils dynamiques et le réutiliser.

**Étapes** :

1. Créer un template “Template CPU Custom”.
2. Ajouter une macro `{$CPU.THRESHOLD}` dans le template.
3. Créer un item `system.cpu.load[percpu,avg1]`.
4. Créer un trigger lié à la macro.
5. Lier ce template à plusieurs hôtes et surcharger la macro au niveau hôte.

---

### **LAB 5 – Dashboard Zabbix et carte réseau**

**Objectif** : Créer un tableau de bord Zabbix et une carte réseau interactive.

**Étapes** :

1. Créer des graphes (CPU, RAM).
2. Créer un dashboard avec widgets :

   * Graphiques
   * Valeurs texte
   * Liste des problèmes
3. Créer une carte réseau avec deux hôtes.
4. Ajouter la carte au dashboard en widget.

---

### **LAB 6 – Supervision SNMP et HTTP**

**Objectif** : Superviser un switch SNMP et vérifier une URL.

**Étapes** :

1. Activer SNMP sur le switch (v2c).
2. Tester depuis le serveur avec `snmpwalk`.
3. Ajouter l’hôte avec interface SNMP dans Zabbix.
4. Lier le template `Generic SNMP`.
5. Créer un item HTTP pour vérifier `http://example.com`.
6. Créer un web scenario avec login + page interne.

---

### **LAB 7 – Découverte automatique d’hôtes et LLD**

**Objectif** : Créer une règle de découverte et utiliser la LLD des interfaces.

**Étapes** :

1. Créer une règle de découverte IP `192.168.1.0/24`.
2. Ajouter une action automatique :

   * Créer l’hôte
   * L’ajouter à un groupe
   * Appliquer un template
3. Vérifier l’apparition de nouveaux hôtes.
4. Explorer la règle LLD "Interface discovery" sur un hôte.
5. Créer un item prototype personnalisé si besoin.

---

### **LAB 8 – Déploiement d’un proxy Zabbix avec TLS**

**Objectif** : Déployer un proxy actif avec chiffrement TLS (PSK).

**Étapes** :

1. Installer `zabbix-proxy` + SQLite sur une autre VM.
2. Générer une clé PSK.
3. Configurer `zabbix_proxy.conf` avec TLS.
4. Déclarer le proxy dans l’interface Zabbix.
5. Démarrer le service proxy.
6. Ajouter un hôte supervisé par ce proxy.

---

### **LAB 9 – Automatisation via API REST de Zabbix**

**Objectif** : Ajouter un hôte via API Zabbix avec Python.

**Étapes** :

1. Écrire un script Python avec `requests`.
2. Authentification à l’API (`user.login`).
3. Création d’un hôte (`host.create`).
4. Création d’un item (`item.create`).
5. Lecture de tous les hôtes (`host.get`).
6. Ajouter une boucle pour créer plusieurs hôtes depuis un CSV.

---

### **LAB 10 – Dashboard Grafana avec données Zabbix**

**Objectif** : Visualiser les données Zabbix dans Grafana.

**Étapes** :

1. Installer Grafana sur une VM.
2. Installer le plugin Zabbix :

   * `grafana-cli plugins install alexanderzobnin-zabbix-app`
3. Ajouter Zabbix comme datasource (`api_jsonrpc.php`).
4. Créer un dashboard :

   * Panel CPU
   * Panel RAM
   * Table disque
5. Ajouter une variable `$host` dynamique.
6. Créer une alerte Grafana (>= 80 % CPU).
7. Exporter le dashboard ou le partager via lien.
