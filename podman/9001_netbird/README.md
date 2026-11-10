# README.md
# Netbird Deployment Guide

## Overview

Netbird is a tool called an Overlay VPN, it is similar to TailScale or ZeroTier but it runs on your server. This file set is highly complex and should not be used by beginers.

Only run this app on it's own dedicated virtual machine, I recommened Ubuntu Server. Do not install any other apps alongside this tool.

This tool requires a router with a public IP that you can open port 443 on. This carries a lot of risk so be careful.

---

## 1. Deployment Prerequisites

Before beginning:
- Create a dedicated VLAN for the publically exposed Netbird server to run in
  - Install qemu-guest-agent
- Create a new Ubuntu Server installation
  - Default settings
- Follow the repository guide to clone the CherryTech GitHub repository.
  - git clone https://github.com/CherryTechLLC/CherryTech-App-Configs.git
- Ensure Podman and Quadlet are installed.

## 2. Configure Caddy

For this application we will set up Caddy **Without** hetzner DNS or a Hetzner API key.

In the Hetzner Console create a DNS A record that will point to your netbird server via your routers public IP

Install the Caddy package and Go. The Caddy package will establish the service, system user, and default configuration directories.

```bash
sudo apt-get install caddy golang -y
```

Copy the complete netbird.Caddyfile into the config directory:

```bash
sudo nano /etc/caddy/Caddyfile
```

Change the domain name on the overlay Caddyfile Entry to your domain

Ensure the domain in the Caddyfile matches the domain configured in your environment file.

Enable Changes:

```bash
sudo systemctl restart caddy
```

## 3. Configure Server Firewall

Allow SSH and HTTPS, deny incoming connections by default, and allow outgoing connections.

```bash
sudo ufw allow 22/tcp

sudo ufw allow 80/tcp

sudo ufw allow 443/tcp

sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw default deny routed
```

Enable UFW:

```bash
sudo ufw enable
```

Verify the firewall configuration:

```bash
sudo ufw status verbose
```

We will configure the router public firewall later

## 3. Create a Dedicated Application User

Create a dedicated system user for netbird:

```bash
sudo useradd -m netbird -F -u 9001
```

Enable lingering so the user's systemd services can run without an active login session:

```bash
sudo loginctl enable-linger netbird
```

To make *sudo -u netbird -i* work better set the account to use bin/bash rather than /bin/sh

```bash
sudo chsh -s /bin/bash netbird
```

## 4. Create and Configure Secrets

### 4.1 Create the Secrets Directory

```bash
sudo mkdir -p /srv/CherryTech-App-Configs/podman/9001_netbird/secrets
```

### 4.2 Generate Passwords

Generate each secure password:

 - Database Password:
```bash
sudo openssl rand -base64 64
```

 - Auth Secret:
```bash
sudo openssl rand -base64 128
```

### 4.3 Create the password files:

Paste the generated password into the file by itself, without additional text or spaces.

```bash
cd /srv/CherryTech-App-Configs/podman/9001_netbird/secrets
```
```bash
sudo nano .netbird-db-key.txt
```
```bash
sudo nano .netbird-auth-secret.txt
```

## 4.4 Create netbird-store-dns:

```bash
sudo nano .netbird-store-dsn.txt
```
Paste the following string and copy the netbird-db-password after *password=*

```bash
host=netbird-db user=netbird password= dbname=netbird port=5432
```

### 4.5 Protect the Secrets

**Mandatory:** Restrict access to the secrets directory and its contents.

```bash
sudo chown -R netbird:netbird /srv/CherryTech-App-Configs/podman/9001_netbird/secrets

sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/9001_netbird/secrets
```

### 4.6 Create the Podman Secret

Log in as the netbird application user:

```bash
sudo -u netbird -i
```

Create the Podman secrets:

```bash
cd /srv/CherryTech-App-Configs/podman/9001_netbird/secrets
```

```bash
podman secret create netbird-db-password .netbird-db-password.txt

podman secret create netbird-auth-key .netbird-auth-key.txt

podman secret create netbird-store-dsn .netbird-store-dsn.txt
```

## 5. Create Application Storage Directories

While logged in as the netbird application user, create the required storage directories:

```bash
mkdir -p /home/netbird/.data/netbird/{netbird-data,netbird-db,netbird-db-backups}
```

## 6. Configure Dashboard Environment Variables

Copy the provided environment file into the application's data directory:

```bash
cp /srv/CherryTech-App-Configs/podman/9001_netbird/dashboard.env /home/netbird/.data/netbird/dashboard.env
```

Edit the environment file:

```bash
nano /home/netbird/.data/netbird/dashboard.env
```

Update the following values:

| Variable | Description |
|---|---|
| `NETBIRD_MGMT_API_ENDPOINT` | Your netbird API domain (e.g., `https://overlay.cherrytech.net`) |
| `NETBIRD_MGMT_GRPC_API_ENDPOINT` | Your netbird API domain (e.g., `https://netbird.apps.cherrytech.net`) |
| `AUTH_AUTHORITY` | Your server's interanal idp (e.g., `https://overlay.cherrytech.net/oauth2`) |

**Important:** Leave `ADMIN_TOKEN` set to `CHANGE-ME`. We will configure a secure admin token through the web interface after deployment.

## 7. Configure Netbird YAML file

Netbird has a large configuration that requires a few changes.

```bash
cp /srv/CherryTech-App-Configs/podman/9001_netbird/config.yaml.example /home/netbird/.data/netbird/config.yaml
```

```bash
nano /home/netbird/.data/netbird/config.yaml
```

Update the following values:

| Variable | Description |
|---|---|
| `exposedAddress` | Your netbird domain (e.g., `https://overlay.cherrytech.net:443`) |
| `auth > issuer` | Your netbird domain (e.g., `https://overlay.cherrytech.net/oauth2`) |
| `auth > dashboardRedirectURIs` | Your netbird domain (e.g., `https://overlay.cherrytech.net/nb-auth`) |
| `auth > dashboardRedirectURIs` | Your netbird domain (e.g., `https://overlay.cherrytech.net/nb-silent-auth`) |

## 8. Install and Start the Quadlet Services

### 8.1 Navigate to the Configuration Directory

If you are not already logged in as the netbird application user:

```bash
sudo -u netbird -i
```

Navigate to the application configuration directory:

```bash
cd /srv/CherryTech-App-Configs/podman/9001_netbird
```

### 8.2 Install the Quadlet Files

Install the application containers and pod:

```bash
podman quadlet install netbird-app.container netbird-server.container netbird-db.container netbird-redis.container netbird.pod netbird.network
```

### 8.3 Configure the Systemd Environment

This command makes the netbird user you logged in as able to execute systemd --user commands:

```bash
export XDG_RUNTIME_DIR="/run/user/$UID"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
```

### 8.4 Reload Systemd and Start netbird

Reload the user systemd configuration:

```bash
systemctl --user daemon-reload
```

Alternatively, from an administrative account:

```bash
sudo systemctl --machine=netbird@.host --user daemon-reload
```

Start the netbird pod:

```bash
systemctl --user start netbird-pod.service
```

Alternatively:

```bash
sudo systemctl --machine=netbird@.host --user start netbird-pod.service
```

### 8.5 Enable Automatic Updates

```bash
systemctl --user enable --now podman-auto-update.timer
```


## 9. Resource Limits and Monitoring

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
sudo journalctl _UID=9001
```
---

## 10. Reference Links

### Container Image
- https://hub.docker.com/r/netbirdio/netbird

### Official Documentation
- https://docs.netbird.io/selfhosted/maintenance/configuration-files
- https://docs.netbird.io/get-started/install

### Additional Resources
- https://hub.docker.com/_/postgres
