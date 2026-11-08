# README.md
# Opsi Deployment Guide

## Overview

OPSI is a tool for deploying applications and configurations to windows fleets. A agent is required on any machine you want to control with OPSI.

A standalone "configserver" has a built-in "depotserver" but additional "depotserver" nodes can be configured in remote campuses to save on bandwidth from the central location.

---

## 1. Deployment Prerequisites

Before beginning:
- Follow the repository guide to clone the CherryTech GitHub repository.
- Ensure Podman and Quadlet are installed.
- Ensure Caddy is installed and running.

## 2. Configure Caddy

Copy the opsi opsi.Caddyfile entry into your server's existing configuration:

```bash
sudo nano /etc/caddy/Caddyfile
```

Place the opsi entry below all existing site entries.

Ensure the domain in the Caddyfile matches the domain configured in your environment file.

## 3. Create a Dedicated Application User

Create a dedicated system user for opsi:

```bash
sudo useradd -m opsi -F -u 6002
```

Enable lingering so the user's systemd services can run without an active login session:

```bash
sudo loginctl enable-linger opsi
```

## 4. Create and Configure Secrets

### 4.1 Create the Secrets Directory

```bash
sudo mkdir -p /srv/CherryTech-App-Configs/podman/6002_opsi/secrets
```

### 4.2 Generate Passwords

Generate each secure password:

 - Database Password:
```bash
sudo openssl rand -base64 64
```

 - MariaDB Root Password:
```bash
sudo openssl rand -base64 64
```

 - Opsi Admin Password:
```bash
sudo openssl rand -base64 32
```

 - Opsi Root Password:
```bash
sudo openssl rand -base64 64
```

 - Opsi Grafana Password:
```bash
sudo openssl rand -base64 64
```

### 4.3 Create the password files:

```bash
cd /srv/CherryTech-App-Configs/podman/6002_opsi/secrets
```
```bash
nano opsi-db-password.txt
```
```bash
nano opsi-db-root-password.txt
```
```bash
nano opsi-admin-password.txt
```
```bash
nano opsi-root-password.txt
```
```bash
nano opsi-grafana-password.txt
```

Paste the generated password into the file by itself, without additional text or spaces.

### 4.4 Protect the Secrets

**Mandatory:** Restrict access to the secrets directory and its contents.

```bash
sudo chown -R opsi:opsi /srv/CherryTech-App-Configs/podman/6002_opsi/secrets

sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/6002_opsi/secrets
```

### 4.5 Create the Podman Secret

Log in as the opsi application user:

```bash
sudo -u opsi -i
```

Create the Podman secrets:

```bash
cd /srv/CherryTech-App-Configs/podman/6002_opsi/secrets
```

```bash
podman secret create opsi-db-password opsi-db-password.txt
```

```bash
podman secret create opsi-db-root-password opsi-db-root-password.txt
```

```bash
podman secret create opsi-admin-password opsi-admin-password.txt
```

```bash
podman secret create opsi-root-password opsi-root-password.txt
```

```bash
podman secret create opsi-grafana-password opsi-grafana-password.txt
```

## 5. Create Application Storage Directories

While logged in as the opsi application user, create the required storage directories:

```bash
mkdir -p /home/opsi/.data/opsi/{opsi-app,opsi-db,opsi-db-backups,opsi-valkey,opsi-grafana-data}
```

## 6. Configure Environment Variables

Copy the provided environment file into the application's data directory:

```bash
cp /srv/CherryTech-App-Configs/podman/6002_opsi/opsi.env /home/opsi/.data/opsi/.env
```

Edit the environment file:

```bash
nano /home/opsi/.data/opsi/.env
```

Update the following values:

| Variable | Description |
|---|---|
| `OPSI_HOSTNAME` | Your opsi domain (e.g., `opsi.apps.cherrytech.net`) |
| `DOMAINNAME` | Your opsi domain again (e.g., `opsi.apps.cherrytech.net`) |
| `TZ` | Your server's timezone (e.g., `America/Detroit`) |


## 7. Install and Start the Quadlet Services

### 7.1 Navigate to the Configuration Directory

If you are not already logged in as the opsi application user:

```bash
sudo -u opsi -i
```

Navigate to the application configuration directory:

```bash
cd /srv/CherryTech-App-Configs/podman/6002_opsi
```

### 7.2 Install the Quadlet Files

Install the application containers and pod:

```bash
podman quadlet install opsi-app.container opsi-grafana.container opsi-db.container opsi-redis.container opsi.pod
```

### 7.3 Configure the Systemd Environment

This command makes the opsi user you logged in as able to execute systemd --user commands:

```bash
export XDG_RUNTIME_DIR="/run/user/$UID"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
```

### 7.4 Reload Systemd and Start opsi

Reload the user systemd configuration:

```bash
systemctl --user daemon-reload
```

Alternatively, from an administrative account:

```bash
sudo systemctl --machine=opsi@.host --user daemon-reload
```

Start the opsi pod:

```bash
systemctl --user start opsi-pod.service
```

Alternatively:

```bash
sudo systemctl --machine=opsi@.host --user start opsi-pod.service
```

**Note:** Full application initialization can take approximately 5–10 minutes before the web interface becomes available.

### 7.5 Reload Enable Automatic Updates

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
- https://hub.docker.com/r/uibmz/opsi-server

### Official Documentation
- https://docs.opsi.org/opsi-docs-en/4.3/first-steps/server/server-installation-docker.html

### Additional Resources
- https://hub.docker.com/_/mariadb
- https://hub.docker.com/r/valkey/valkey
- https://hub.docker.com/r/grafana/grafana
- https://github.com/opsi-org/opsi-docker/blob/main/opsi-server/docker-compose.yml
- https://github.com/opsi-org/opsi-docker/blob/main/opsi-server/opsi-server.env
