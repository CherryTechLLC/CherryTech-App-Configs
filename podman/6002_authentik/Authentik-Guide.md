Authentik is a tool designed to provide strong user account control (UAC) for internally hosted applications.

DEPLOYMENT INSTRUCTIONS:
  - Create a fresh installation of Ubuntu Server
    - Ensure you have a very strong password
    - Update system
      - sudo apt update
      - sudo apt dist-upgrade
      - sudo apt autoremove
  - Install required packages
    - sudo apt-get install qemu-guest-agent git podman
      - Skip qemu-guest-agent if not using proxmox
  - Set up ubuntu server automatic updates
    - 
  - Create a new email account at your email service provider
    - sso@example.com
    - Save the password to put into a podman secret later
  - Follow Caddy set up guide
  - Copy the Caddyfile entry into your servers existing /etc/caddy/Caddyfile below all other entries
  - Create a new dedicated user for this app
    - sudo useradd authentik -m -F -u 6002
    - sudo loginctl enable-linger authentik
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
    - Files: (Paste text into file by itself)
      - sudo nano .authentik-db-password.txt
        - sudo openssl rand -base64 32
      - sudo nano .authentik-secret-key.txt
        - sudo openssl rand -base64 64
      - sudo nano .authentik-email-password.txt
        - Enter
    - Protect Secrets (MANDATORY)
      - sudo chown -R authentik:authentik /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
    - Generate Secrets
      - Login to app user (sudo -u authentik -i)
      - podman secret create authentik-db-password .authentik-db-password.txt
      - podman secret create authentik-secret-key .authentik-secret-key.txt
  - Create storage directories: (sudo -u authentik -i)
    - mkdir /home/authentik/.data/authentik/authentik-db
    - mkdir /home/authentik/.data/authentik/authentik-db-backups
    - mkdir /home/authentik/.data/authentik/authentik-app
    - mkdir /home/authentik/.data/authentik/authentik-custom-templates
    - mkdir /home/authentik/.data/authentik/authentik-certs
  - Configure Environment Variable
    - cp /srv/CherryTech-App-Configs/podman/6002_authentik/authentik.env /home/authentik/.data/authentik/.env
    - Values to change:
      - AUTHENTIK_EMAIL__HOST (Your server provider, mail.your-server.de for Hetzner)
      - AUTHENTIK_EMAIL__PORT (465 for ssl, 587 for tls)
      - AUTHENTIK_EMAIL__USERNAME (The part of the email before @)
      - Uncomment AUTHENTIK_EMAIL__USE_TLS - OR - AUTHENTIK_EMAIL__USE_SSL (Not Both!)
      - AUTHENTIK_EMAIL__FROM (sso@example.com)
  - Create SystemD Quadlet
    - sudo -u authentik -i
    - cd /srv/CherryTech-App-Configs/podman/6002_authentik
    - podman quadlet install authentik-app.container authentik-db.container authentik-worker.container authentik.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start authentik-pod.service
  - Full app initialization can take 5-10 minutes before the app will display
  - Configure Automatic Updates
    - systemctl --user enable --now podman-auto-update.timer

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  - https://docs.goauthentik.io/
Other Good Sources:
  - https://github.com/goauthentik/authentik
  - https://hub.docker.com/_/postgres

Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
Most Postgres and MariaDB passwords can be long random passwords. Some systems however require specific charactors sets and requirements
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
