# README.md
# EXAMPLE Deployment Guide

## Overview



---

## 1. Deployment Prerequisites

Before beginning:
- Follow the repository guide to clone the CherryTech GitHub repository.
- Ensure Podman and Quadlet are installed.
- Ensure Caddy is installed and running.

## 2. Configure Caddy

Copy the EXAMPLE EXAMPLE.Caddyfile entry into your server's existing configuration:

```bash
sudo nano /etc/caddy/Caddyfile
```

Place the EXAMPLE entry below all existing site entries.

Ensure the domain in the Caddyfile matches the domain configured in your environment file.

## 3. Create a Dedicated Application User

Create a dedicated system user for EXAMPLE:

```bash
sudo useradd -m EXAMPLE -F -u 0000
```

Enable lingering so the user's systemd services can run without an active login session:

```bash
sudo loginctl enable-linger EXAMPLE
```

## 4. Create and Configure Secrets

### 4.1 Create the Secrets Directory

```bash
sudo mkdir -p /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
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

### 4.3 Create the password file:

```bash
cd /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
```
```bash
sudo nano .EXAMPLE-db-password.txt
```
```bash
sudo nano .EXAMPLE-db-root-password.txt
```

Paste the generated password into the file by itself, without additional text or spaces.

### 4.4 Protect the Secrets

**Mandatory:** Restrict access to the secrets directory and its contents.

```bash
sudo chown -R EXAMPLE:EXAMPLE /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets

sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
```

### 4.5 Create the Podman Secret

Log in as the EXAMPLE application user:

```bash
sudo -u EXAMPLE -i
```

Create the Podman secrets:

```bash
cd /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
```

```bash
podman secret create EXAMPLE-db-password .EXAMPLE-db-password.txt
```

```bash
podman secret create EXAMPLE-db-root-password .EXAMPLE-db-root-password.txt
```

## 5. Create Application Storage Directories

While logged in as the EXAMPLE application user, create the required storage directories:

```bash
mkdir -p /home/EXAMPLE/.data/EXAMPLE/{EXAMPLE-app,EXAMPLE-data,EXAMPLE-db,EXAMPLE-db-backups,EXAMPLE-valkey}
```

## 6. Configure Environment Variables

Copy the provided environment file into the application's data directory:

```bash
cp /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/EXAMPLE.env /home/EXAMPLE/.data/EXAMPLE/.env
```

Edit the environment file:

```bash
nano /home/EXAMPLE/.data/EXAMPLE/.env
```

Update the following values:

| Variable | Description |
|---|---|
| `DOMAIN` | Your EXAMPLE domain (e.g., `EXAMPLE.apps.cherrytech.net`) |
| `TZ` | Your server's timezone (e.g., `America/Detroit`) |

**Important:** Leave `ADMIN_TOKEN` set to `CHANGE-ME`. We will configure a secure admin token through the web interface after deployment.

## 7. Install and Start the Quadlet Services

### 7.1 Navigate to the Configuration Directory

If you are not already logged in as the EXAMPLE application user:

```bash
sudo -u EXAMPLE -i
```

Navigate to the application configuration directory:

```bash
cd /srv/CherryTech-App-Configs/podman/0000_EXAMPLE
```

### 7.2 Install the Quadlet Files

Install the application containers and pod:

```bash
podman quadlet install EXAMPLE-app.container EXAMPLE-server.container EXAMPLE-db.container EXAMPLE-redis.container EXAMPLE.pod EXAMPLE.network
```

### 7.3 Configure the Systemd Environment

This command makes the EXAMPLE user you logged in as able to execute systemd --user commands:

```bash
export XDG_RUNTIME_DIR="/run/user/$UID"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
```

### 7.4 Reload Systemd and Start EXAMPLE

Reload the user systemd configuration:

```bash
systemctl --user daemon-reload
```

Alternatively, from an administrative account:

```bash
sudo systemctl --machine=EXAMPLE@.host --user daemon-reload
```

Start the EXAMPLE pod:

```bash
systemctl --user start EXAMPLE-pod.service
```

Alternatively:

```bash
sudo systemctl --machine=EXAMPLE@.host --user start EXAMPLE-pod.service
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
sudo journalctl _UID=0000
```
---

## 11. Reference Links

### Container Image
-

### Official Documentation
-

### Additional Resources
- https://hub.docker.com/_/mariadb
- https://hub.docker.com/_/postgres
- https://hub.docker.com/r/valkey/valkey
