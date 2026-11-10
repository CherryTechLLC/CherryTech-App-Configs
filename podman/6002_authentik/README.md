# README.md
# Authentik Deployment Guide

## Overview

Authentik is a tool designed to provide strong user account control (UAC) for internally hosted applications. Authentik asks as and identity provider (idp) and single sign on (SSO).

---

## 1. Deployment Prerequisites

Before beginning:
- Follow the repository guide to clone the CherryTech GitHub repository.
- Ensure Podman and Quadlet are installed.
- Ensure Caddy is installed and running.
- Have an SMTP account available for email configuration.
  - Will be placed in the .authentik-email-password.txt file so it can be converted to a secret

## 2. Configure Caddy

Copy the authentik authentik.Caddyfile entry into your server's existing configuration:

```bash
sudo nano /etc/caddy/Caddyfile
```

Copy the authentik entry in, below all existing site entries.

Ensure the domain in the Caddyfile matches the domain configured in your environment file.

Enable Changes:

```bash
sudo systemctl restart caddy
```

## 3. Create a Dedicated Application User

Create a dedicated system user for authentik:

```bash
sudo useradd -m authentik -F -u 6002
```

Enable lingering so the user's systemd services can run without an active login session:

```bash
sudo loginctl enable-linger authentik
```

To make sudo -u authentik -i work better set the account to use bin/bash rather than /bin/sh

```bash
sudo chsh -s /bin/bash authentik
```

## 4. Create and Configure Secrets

### 4.1 Create the Secrets Directory

```bash
sudo mkdir -p /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
```

### 4.2 Generate Passwords

Generate each secure password:

Database Password:
```bash
sudo openssl rand -base64 64
```

Secret Key:
```bash
sudo openssl rand -base64 64
```

### 4.3 Create the password file:

Paste the generated password into the file by itself, without additional text or spaces.

```bash
cd /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
```
```bash
sudo nano .authentik-db-password.txt
```
```bash
sudo nano .authentik-secret-key.txt
```

Paste the authentik SMTP password file in by itself without additional text or spaces.

```bash
sudo nano .authentik-email-password.txt
```

### 4.4 Protect the Secrets

**Mandatory:** Restrict access to the secrets directory and its contents.

```bash
sudo chown -R authentik:authentik /srv/CherryTech-App-Configs/podman/6002_authentik/secrets

sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
```

### 4.5 Create the Podman Secret

Log in as the authentik application user:

```bash
sudo -u authentik -i
```

Create the Podman secrets:

```bash
cd /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
```

```bash
podman secret create authentik-db-password .authentik-db-password.txt

podman secret create authentik-secret-key .authentik-secret-key.txt

podman secret create authentik-email-password .authentik-email-password.txt
```

## 5. Create Application Storage Directories

While logged in as the authentik application user, create the required storage directories:

```bash
mkdir -p /home/authentik/.data/authentik/authentik-app
mkdir -p /home/authentik/.data/authentik/authentik-db
mkdir -p /home/authentik/.data/authentik/authentik-db-backups
mkdir -p /home/authentik/.data/authentik/authentik-custom-templates
mkdir -p /home/authentik/.data/authentik/authentik-certs
```

## 6. Configure Environment Variables

Copy the provided environment file into the application's data directory:

```bash
cp /srv/CherryTech-App-Configs/podman/6002_authentik/authentik.env /home/authentik/.data/authentik/.env
```

Edit the environment file:

```bash
nano /home/authentik/.data/authentik/.env
```

Update the following values:

| Variable | Description |
|---|---|
| `AUTHENTIK_WEB__BASE_URL` | Your authentik domain (e.g., `https://sso.cherrytech.net`) |
| `AUTHENTIK_EMAIL__HOST` | Your email servers addess (e.g., `mail.your-server.de`) (Hetzner) |
| `AUTHENTIK_EMAIL__PORT` | Your email server MX port (e.g., `465 (SSL) / 587 (STARTTLS)`) |
| `AUTHENTIK_EMAIL__USERNAME` | Your authentik email account username (e.g., `sso`) |
| `AUTHENTIK_EMAIL__USE_TLS` | Uncomment if using TLS for email (e.g., `true`) |
| `AUTHENTIK_EMAIL__USE_SSL` | Uncomment if using SSL for email (e.g., `true`) |
| `AUTHENTIK_EMAIL__FROM` | Uncomment if using SSL for email (e.g., `sso@cherrytech.net`) |
| `TZ` | Your server's timezone (e.g., `America/Detroit`) |


**Important:** Leave `ADMIN_TOKEN` set to `CHANGE-ME`. We will configure a secure admin token through the web interface after deployment.

## 7. Install and Start the Quadlet Services

### 7.1 Navigate to the Configuration Directory

If you are not already logged in as the authentik application user:

```bash
sudo -u authentik -i
```

Navigate to the application configuration directory:

```bash
cd /srv/CherryTech-App-Configs/podman/6002_authentik
```

### 7.2 Install the Quadlet Files

Install the application containers and pod:

```bash
podman quadlet install authentik-app.container authentik-db.container authentik-worker.container authentik.pod
```

### 7.3 Configure the Systemd Environment

This command makes the authentik user you logged in as able to execute systemd --user commands:

```bash
export XDG_RUNTIME_DIR="/run/user/$UID"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
```

### 7.4 Reload Systemd and Start authentik

Reload the user systemd configuration:

```bash
systemctl --user daemon-reload
```

Alternatively, from an administrative account:

```bash
sudo systemctl --machine=authentik@.host --user daemon-reload
```

Start the authentik pod:

```bash
systemctl --user start authentik-pod.service
```

Alternatively:

```bash
sudo systemctl --machine=authentik@.host --user start authentik-pod.service
```

### 7.5 Enable Automatic Updates

```bash
systemctl --user enable --now podman-auto-update.timer
```


## 8. Resource Limits and Monitoring

The configured resource limits are intended to prevent application resource-management issues from causing a full server crash.

The provided values are designed for approximately **5–10 users**.

If you anticipate supporting more users, or notice resource utilization approaching the configured limits, consider increasing the allocations.

### Monitoring

Use the CentOS Cockpit interface to monitor application resource utilization:

`https://CENTOS-SERVER-IP:9090`

Monitor:
- CPU utilization
- Memory utilization
- Container resource consumption

If utilization consistently approaches the configured limits, adjust the corresponding Quadlet resource allocations.

To view logs go the the **Logs** tab in cockpit and under the **Identifier** dropdown select the container you want to view logs on.

To view full logs for the application log into an admin account and use the following command:

```bash
sudo journalctl _UID=6002
```
---

## 11. Reference Links

### Container Image
- https://hub.docker.com/r/authentik/server

### Official Documentation
- https://docs.goauthentik.io/

### Additional Resources
- https://hub.docker.com/_/postgres
- https://github.com/goauthentik/authentik

