# Debian VM Initial Setup – Ansible Playbook

## Ziel dieses Repos
Dieses Playbook soll alle Tools installieren und alle Einstellungen setzen, die ich auf einer Debian VM brauche.

### Info zur VM installation
Die VM wurde bereits mit dem Standard installationsverfahren installiert. (Hostname wurde vergeben, User wurde erstellt, etc)

### Was das Playbook umsetzt:
1. SSH Key für initialen User auf Zielsystem hinterlegen
2. System Update durchführen 
3. User konfiguration übernehmen
4. SSH Hardening vornehmen
5. ZSH installieren und als standard Shell einrichten
6. Docker installieren
7. Minikube installieren

## Projektstruktur

```
.
├── site.yml                          # Haupt-Playbook
├── group_vars/all/
│   ├── main.yml                      # Allgemeine Variablen (anpassen!)
│   └── vault.yml                     # Secrets (verschlüsseln!)
├── inventory/inventory.ini
└── roles/
    ├── system-update/tasks/main.yml  # apt update/upgrade + Basispakete
    ├── user-setup/tasks/main.yml     # Benutzer erstellen + sudo + SSH Key
    ├── ssh-hardening/
    │   ├── tasks/main.yml            # SSH Key, Root-Login, PW-Auth
    │   ├── templates/sshd_config.j2  # SSH-Konfiguration
    │   └── handlers/main.yml         # SSH Restart Handler
    ├── docker/tasks/main.yml         # Docker CE Installation
    ├── minikube/tasks/main.yml       # kubectl + Minikube
    └── zsh-setup/
        ├── tasks/main.yml            # ZSH + Oh-My-Zsh + Plugins
        └── templates/zshrc.j2        # .zshrc Konfiguration
```

## Vorbereitung

### 1. Variablen ggf. anpassen 
```bash
# group_vars/all/main.yml bearbeiten
nano group_vars/all/main.yml
```

### 2. Vault für Secrets anlegen
```bash
# vault.yml bearbeiten und dann verschlüsseln
nano group_vars/all/vault.yml
ansible-vault encrypt group_vars/all/vault.yml
```

### 3. IPs in inventory file eintragen
```bash
# inventory/inventory.ini bearbeiten
nano inventory/inventory.ini
```

## Ausführen

### Erstmalig (mit Root-Passwort, bevor SSH gehärtet ist)
```bash
ansible-playbook site.yml -i inventory/inventory.ini \
  --ask-pass \
  --ask-become-pass \
  --ask-vault-pass
```

### Nach dem ersten Run (mit SSH-Key des neuen Benutzers)
```bash
ansible-playbook site.yml -i inventory/inventory.ini \
  --private-key ~/.ssh/servers \
  --ask-vault-pass \
  --ask-become-pass
```

### Nur bestimmte Rollen ausführen (Tags)
```bash
# Nur Docker installieren
ansible-playbook site.yml -i inventory.ini --tags docker --ask-vault-pass

# Nur SSH härten
ansible-playbook site.yml -i inventory.ini --tags ssh --ask-vault-pass
```

## Wichtige Hinweise

> **SSH-Härtung**: Nach dem Playbook ist Root-Login und Passwort-Auth deaktiviert.
> Stelle sicher, dass dein SSH-Key korrekt in `vault.yml` hinterlegt ist,
> bevor du das Playbook ausführst – sonst sperrst du dich aus!

> **Vault**: Committe niemals `vars/vault.yml` unverschlüsselt in Git.
> Füge `vars/vault.yml` zu `.gitignore` hinzu oder verschlüssele es immer mit `ansible-vault`.

> **Minikube**: Läuft standardmäßig mit dem Docker-Treiber.
> Der Benutzer muss in der `docker`-Gruppe sein (wird automatisch erledigt).
> Zum Starten: `minikube start --driver=docker`
