# EXAMPLE Deployment Guide

## Overview

OPSI is an open source tool for deploying and managing applications installed on a Windows desktop. 

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

### 4.2 Generate the Database Password

Create the password file:

```bash
cd /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
nano EXAMPLE-db-password.txt
```

Generate a secure password:

```bash
sudo openssl rand -base64 64
```

Paste the generated password into the file by itself, without additional text or spaces.

### 4.3 Protect the Secrets

**Mandatory:** Restrict access to the secrets directory and its contents.

```bash
sudo chown -R EXAMPLE:EXAMPLE /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets

sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
```

### 4.4 Create the Podman Secret

Log in as the EXAMPLE application user:

```bash
sudo -u EXAMPLE -i
```

Create the Podman secret:

```bash
podman secret create EXAMPLE-db-password /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets/EXAMPLE-db-password.txt
```

## 5. Create Application Storage Directories

While logged in as the EXAMPLE application user, create the required storage directories:

```bash
mkdir -p /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-app /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-db /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-db-backups
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
podman quadlet install EXAMPLE-app.container EXAMPLE-db.container EXAMPLE.pod EXAMPLE.network
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

**Note:** Full application initialization can take approximately 5–10 minutes before the web interface becomes available.

## 8. Initial Admin Configuration

Once EXAMPLE has initialized, navigate to:

`https://vault.apps.domain.tld/admin`

Log in using the temporary admin token:

`CHANGE-ME`

### 8.1 Generate a Secure Admin Token

Follow the official [EXAMPLE Admin Page documentation](https://github.com/dani-garcia/EXAMPLE/wiki/Enabling-admin-page).

Install the required packages:

```bash
sudo dnf install epel-release -y

sudo dnf install argon2 -y
```

Generate an Argon2id password hash:

```bash
sudo echo -n 'REPLACE-THIS-PASSWORD' | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4
```

Replace `REPLACE-THIS-PASSWORD` with your desired admin password.

**Important:**
- Save the password somewhere secure.
- Copy the entire generated string, including the `$argon2id$` prefix.
- Do not lose the original password. The generated hash cannot be used as the login password.

### 8.2 Apply the Admin Token

1. Open the EXAMPLE admin panel.
2. Locate **Admin token/Argon2 PHC** under General Settings.
3. Paste the entire generated Argon2id hash.
4. Select **Save** at the bottom of the page.
5. Log out using the top-right menu.
6. Return to the admin login page.
7. Enter the original password used to generate the hash.

## 9. Configure SMTP

SMTP (Simple Mail Transfer Protocol) is required for EXAMPLE email functionality, including invitations and account-related notifications.

Configure the following settings in the admin panel:

| Setting | Description | Example |
|---|---|---|
| SMTP Host | Hostname of your email server | `mail.your-server.de` |
| SMTP From | Email address notifications originate from | `EXAMPLE@example.com` |
| SMTP From Name | Display name shown to recipients | `EXAMPLE` |
| SMTP Username | SMTP authentication username | `EXAMPLE@example.com` |
| SMTP Password | SMTP account password | Your SMTP password |

**Notes:**
- The SMTP hostname depends on your email provider. The example above is for Hetzner.
- The From address must be authorized to send through your SMTP provider.
- The username is generally the full email address.
- Ensure the SMTP port and security settings match your provider's requirements.

Save the configuration and test email delivery.

---

## 10. Resource Limits and Monitoring

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
- [EXAMPLE – Docker Hub](https://hub.docker.com/r/EXAMPLE/server)

### Official Documentation
- [EXAMPLE Wiki](https://github.com/dani-garcia/EXAMPLE/wiki)
- [SMTP Configuration](https://github.com/dani-garcia/EXAMPLE/wiki/SMTP-Configuration)
- [Enabling the Admin Page](https://github.com/dani-garcia/EXAMPLE/wiki/Enabling-admin-page)
- [Using Podman](https://github.com/dani-garcia/EXAMPLE/wiki/Using-Podman)

### Additional Resources
- [EXAMPLE Quadlet / PostgreSQL Pod Guide](https://www.nite07.com/en/posts/quadlet-EXAMPLE/#option-2-postgresql-pod)
