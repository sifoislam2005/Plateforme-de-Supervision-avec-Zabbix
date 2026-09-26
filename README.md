# 📊 Plateforme de Supervision Centralisée avec Zabbix

Ce projet présente la mise en place d'une solution de supervision centralisée (Monitoring) basée sur **Zabbix 7.0 LTS** au sein d'une infrastructure d'entreprise virtualisée composée de zones isolées (LAN et DMZ) sécurisées par un pare-feu **pfSense** et un domaine **Active Directory**.

---

## 🏗️ Architecture Réseau & Matrice des Équipements

Le serveur Zabbix est positionné dans la zone **LAN** (`192.168.1.15/24`) afin de garantir qu'aucune base de données ni interface d'administration ne soit exposée dans la DMZ, tout en franchissant le pare-feu pfSense via des règles strictes pour auditer les services.

| Hôte | Zone Réseau | Adresse IP | Rôle / Description |
| :--- | :--- | :--- | :--- |
| **SRV-ZABBIX** | LAN | `192.168.1.15/24` | Serveur Zabbix 7.0 LTS (Debian 12 + MariaDB + Apache) |
| **SRV-AD** | LAN | `192.168.1.10/24` | Contrôleur de Domaine Windows Server 2022 |
| **CLIENT-W11** | LAN | DHCP via WS | Poste client Windows 11 |
| **SRV-WEB-DEBIAN** | DMZ | `192.168.2.10/24` | Serveur Web DMZ (Debian 12) |
| **VM-PFSENSE** | Passerelle | `192.168.1.1` / `192.168.2.1` | Pare-feu & Routage inter-zones |

---

## 🚀 Déploiement & Configuration

### 1. Installation du Serveur Zabbix (Debian 12)
* **Paquets officiels & Base de données :** Installation de `zabbix-server-mysql`, `zabbix-frontend-php`, `zabbix-apache-conf` et `mariadb-server`.
* **Base MariaDB :** Création de la base `zabbix` et attribution des privilèges à l'utilisateur `zabbix@localhost`.
* **Authentification :** Configuration de `DBPassword` dans `/etc/zabbix/zabbix_server.conf`.

### 2. Filtrage Réseau (pfSense)
* **Règle LAN à DMZ :** Autorisation du protocole **TCP** depuis la source `192.168.1.15` vers la destination `192.168.2.10` sur le port **10050** (Agent Zabbix).

### 3. Déploiement des Agents
* **DMZ (SRV-WEB-DEBIAN) :** Installation de `zabbix-agent` léger et liaison avec le serveur Zabbix via `/etc/zabbix/zabbix_agentd.conf` (`Server=192.168.1.15`).
* **LAN (SRV-AD) :** Installation de **Zabbix Agent 2 (64-bit)** sur Windows Server 2022 et affectation du modèle *Windows by Zabbix agent*.

---

## 🔔 Configuration de la Centralisation des Alertes

### 📬 1. Notifications Telegram (Webhook)
* Bot créé via **BotFather** (`@zabbixxvm_bot`) avec récupération du Chat ID via `@myidbot`.
* Configuration du média **Telegram** dans Zabbix (`api_token`, `api_chat_id`, `api_parse_mode`) et association à l'action d'alerte.

### ✉️ 2. Notifications SMTP (Gmail)
* **Serveur SMTP :** `smtp.gmail.com:587` avec chiffrement **STARTTLS** et mot de passe d'application Google.
* **Résolution d'erreur (`No message defined for media type`) :**
  * Basculement du format de message en **HTML**.
  * Ajout obligatoire des modèles de messages (*Message Templates*) pour les événements **Problème** et **Récupération de problème**.

---

## 🧪 Validation & Tests d'Incidents

1. **Simulation d'incidents :** Arrêt manuel du service `Spouleur d'impression` (*Spooler*) sur `SERVER-AD` (`services.msc`).
2. **Alerting instantané :**
   * Transmission simultanée de la notification d'incident sur le canal **Telegram** et par **E-mail** (Boîte de réception).
   * Détails transmis : Nom de l'hôte (`SERVER-AD`), heure exacte, sévérité (*Average*) et état du service.
3. **Rétablissement :** Envoi automatique de la notification de résolution (*RESOLVED*) dès le redémarrage du service.
