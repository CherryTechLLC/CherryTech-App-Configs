# Caddy Installation – Debian / Ubuntu

These instructions walk through installing Caddy with the **Hetzner DNS module** for automated DNS-01 certificate validation.

## 1. DNS Configuration

In order for the domains assigned to applications to work, you must create a DNS A record with your DNS provider.

I recommend creating a wildcard record for your applications:

```text
*.apps.example.com
```

You can then assign individual subdomains to applications in your Caddy configuration:

```text
your-app.apps.example.com
```

## 2. Install Caddy and Go

Install the Caddy package and Go. The Caddy package will establish the service, system user, and default configuration directories.

```bash
sudo apt-get install caddy golang -y
```

**Note:** The distribution-provided Caddy binary will be replaced with a custom build containing the Hetzner DNS module.

Prevent APT from overwriting the custom Caddy binary during future updates:

```bash
sudo apt-mark hold caddy
```

## 3. Build Caddy with the Hetzner DNS Module

**Important:** Check for a newer version of xcaddy (https://github.com/caddyserver/xcaddy/releases) before running these commands.

Download and extract xcaddy:

```bash
cd /tmp

wget https://github.com/caddyserver/xcaddy/releases/download/v0.4.7/xcaddy_0.4.7_linux_amd64.tar.gz

tar xvf xcaddy_0.4.7_linux_amd64.tar.gz xcaddy

sudo mv xcaddy /usr/bin/
```

Build Caddy with the Hetzner DNS module:

```bash
xcaddy build --with github.com/caddy-dns/hetzner/v2
```

Stop the existing Caddy service and replace its binary:

```bash
sudo systemctl stop caddy

sudo cp /tmp/caddy /usr/bin/caddy
```

Verify the custom build:

```bash
caddy build-info
```

## 4. Configure Caddy Environment Variables

Set the ownership of the Caddy configuration:

```bash
sudo chown caddy:caddy /etc/caddy/Caddyfile
```

Copy the environment template:

```bash
sudo cp /srv/CherryTech-App-Configs/caddy/caddy.env.template /etc/caddy/caddy.env
```

Edit the environment file:

```bash
sudo nano /etc/caddy/caddy.env
```

Configure the following:

- **HETZNER_API_TOKEN:** Create an API token in Hetzner DNS and enter it into this field.
  - Log into *Hetzner Console*
  - Select the project for domain you want the apps to fall under. 
  - On the left side-bad go down to *Security*
  - Go to the *API tokens* tab and select *Generate API token*
  - Configure the token
    - **DO NOT USE THE SAME TOKEN ON MULTIPLE SERVERS**
    - Name the key something specific to this server (e.g "Apps")
    - Set the key to *Read & Write*
    - Copy the new key directly into /etc/caddy/caddy.env
- **DOMAIN:** Set this to the base domain used by your applications, typically `apps.example.com` or `example.com`, depending on your configuration.

Secure the environment file:

```bash
sudo chmod 600 /etc/caddy/caddy.env

sudo chown caddy:caddy /etc/caddy/caddy.env
```

## 5. Configure the Caddy Systemd Service

Create a systemd override to load the environment variables:

```bash
sudo systemctl edit caddy
```

Add the following:

```ini
[Service]
EnvironmentFile=/etc/caddy/caddy.env
```

Save and exit.

Copy your Caddyfile into the configuration directory:

```bash
sudo cp /srv/CherryTech-App-Configs/caddy/Caddyfile /etc/caddy/Caddyfile
```

## 6. Configure the Firewall

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

## 7. Start Caddy

Reload systemd to apply the service override:

```bash
sudo systemctl daemon-reload
```

Enable and start Caddy:

```bash
sudo systemctl enable --now caddy
```

Verify the service:

```bash
sudo systemctl status caddy
```

## 8. Maintenance

Due to the utilization of a custom Caddy application with the Hetzner-DNS module updates to Caddy will require rebuilding the custom binary as it was built in step 3.

Download and extract xcaddy: (**Check for a new version**)

```bash
cd /tmp

wget https://github.com/caddyserver/xcaddy/releases/download/v0.4.7/xcaddy_0.4.7_linux_amd64.tar.gz

tar xvf xcaddy_0.4.7_linux_amd64.tar.gz xcaddy

sudo mv xcaddy /usr/bin/
```

Build Caddy with the Hetzner DNS module:

```bash
xcaddy build --with github.com/caddy-dns/hetzner/v2
```

Stop the existing Caddy service and replace its binary:

```bash
sudo systemctl stop caddy

sudo cp /tmp/caddy /usr/bin/caddy
```

Verify the custom build:

```bash
caddy build-info
```
