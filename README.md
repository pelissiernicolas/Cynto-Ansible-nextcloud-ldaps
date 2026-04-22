# Cynto - Déploiement Nextcloud avec LDAP via Ansible

## Présentation

Ce projet permet le déploiement automatisé d’une plateforme Nextcloud intégrée à un annuaire Active Directory via LDAP, à l’aide d’Ansible.

Il s’inscrit dans une infrastructure plus globale et peut s’intégrer avec le projet :

- Cynto-Ansible-AD : déploiement de l’Active Directory

Ce projet automatise :

- L’installation de Nextcloud
- La configuration de MariaDB
- L’intégration LDAP avec Active Directory
- La configuration des services complémentaires (AppAPI, HaRP, HAProxy)
- La gestion sécurisée des secrets via Ansible Vault
## Prérequis

### Côté Ansible

- Ansible installé
- Accès SSH au serveur Nextcloud (Linux)
- Accès WinRM aux serveurs Active Directory (Windows)
### Côté infrastructure

- Serveur Nextcloud (Debian / Ubuntu recommandé)
- Serveur Active Directory opérationnel
## Préparation des serveurs

### Serveur Nextcloud (Linux)

- Accès SSH fonctionnel
- Compte avec privilèges sudo

### Serveurs Active Directory

Configurer WinRM (si non déjà fait via le projet AD) :

```Powershell
winrm quickconfig  
Enable-PSRemoting -Force  
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true  
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true  
Enable-NetFirewallRule -Name *WinRM*
```
## Structure du projet

```bash
Cynto-Ansible-nextcloud-ldaps/  
├── inventories/  
│   └── prod/  
│       ├── group_vars/  
│       │   ├── nextcloud/  
│       │   │   └── vault.yml  
│       │   └── ad/  
│       │       └── vault.yml  
├── playbooks/  
├── roles/  
└── README.md
```
## Gestion des secrets avec Ansible Vault

Les mots de passe et informations sensibles sont stockés dans deux fichiers Vault distincts :

- Un pour Nextcloud
- Un pour Active Directory
### 1. Création du Vault Nextcloud

```bash
ansible-vault create inventories/prod/group_vars/nextcloud/vault.yml
```

Contenu :

```YML
vault_nextcloud_admin_user: ncadmin  
vault_nextcloud_admin_password: "xxx"  
vault_nextcloud_db_password: "xxx"  
vault_mariadb_root_password: "xxx"  
vault_appapi_haproxy_password: "xxx"  
vault_harp_shared_key: "xxx"  
vault_ldap_bind_password: "xxx"  
vault_ad_admin_password: "xxx"  
vault_service_account_password: "xxx"
```

### 2. Création du Vault Active Directory

```bash
ansible-vault create inventories/prod/group_vars/ad/vault.yml
```

Contenu :

```YML
vault_windows_admin_password: "xxx"  
vault_service_account_password: "xxx"
```
### 3. Consultation des Vault

```bash
ansible-vault view inventories/prod/group_vars/nextcloud/vault.yml  
ansible-vault view inventories/prod/group_vars/ad/vault.yml
```
### 4. Modification

```
ansible-vault edit inventories/prod/group_vars/nextcloud/vault.yml  
ansible-vault edit inventories/prod/group_vars/ad/vault.yml
```
## Configuration de l’inventaire

Exemple :

```INI
[nextcloud]  
srv-nextcloud-01 ansible_host=10.8.40.13  
  
[ad]  
srv-ad-01 ansible_host=10.8.40.10  
srv-ad-02 ansible_host=10.8.40.11
```
## Vérification des connexions

### Test SSH (Nextcloud)

```bash
ansible nextcloud -m ping
```
### Test WinRM (AD)

```bash
ansible ad -m win_ping
```
## Déploiement

```bash
ansible-playbook playbooks/00-site.yml --ask-vault-pass --ask-pass --ask-become-pass
```
## Intégration LDAP

Le projet configure automatiquement :

- Connexion LDAP vers Active Directory
- Compte de service pour le bind LDAP
- Synchronisation des utilisateurs AD dans Nextcloud

Pré-requis côté AD :
- Compte de service LDAP
- Droits de lecture sur l’annuaire
## Sécurité

- Ne pas stocker de secrets en clair
- Utiliser Ansible Vault systématiquement
- En production :
    - Utiliser LDAPS
    - Utiliser HTTPS pour Nextcloud
    - Restreindre les accès réseau
## Dépendances fonctionnelles

Ce projet dépend d’une infrastructure Active Directory fonctionnelle.
Il est recommandé de déployer en amont :

- Le projet Cynto-Ansible-AD
## Maintenabilité

- Séparation des variables par rôle
- Gestion centralisée des secrets
- Architecture modulaire Ansible
- Possibilité d’extension (HA, multi-site, scaling)
## Auteur

Nicolas Pelissier
