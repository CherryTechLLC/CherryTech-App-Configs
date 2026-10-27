Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Create a new dedicated user for this app
    - sudo useradd -m inventree -F -u 8016
    - sudo loginctl enable-linger inventree
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
    - Files: (Paste text into file by itself)
      - sudo nano .inventree-db-password.txt
        - sudo openssl rand -base64 64
      - sudo nano .inventree-secret-key.txt
        - sudo openssl rand -base64 128
      - sudo nano .inventree-admin-password.txt
        - sudo openssl rand -base64 32
      - sudo nano .inventree-
    - Protect Secrets (MANDATORY)
      - sudo chown -R inventree:inventree /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
    - Generate Secrets
      - Login to app user (sudo -u inventree -i)
      - podman secret create inventree-db-password .inventree-db-password.txt
      - podman secret create inventree-secret-key .inventree-secret-key.txt
      - podman secret create inventree-secret-key .inventree-admin-password.txt
  - Create storage directories: (sudo -u inventree -i)
    - mkdir /home/inventree/.data/inventree/inventree-db
    - mkdir /home/inventree/.data/inventree/inventree-db-backups
    - mkdir /home/inventree/.data/inventree/inventree-valkey
    - mkdir /home/inventree/.data/inventree/inventree-app
    - mkdir /home/inventree/.data/inventree/inventree-config
    - mkdir /home/inventree/.data/inventree/inventree-data
    - cp /srv/CherryTech-App-Configs/podman/8016_inventree/Caddyfile.internal /home/inventree/.data/inventree/Caddyfile
  - Configure Environment Variable (sudo -u inventree -i)
    - cp /srv/CherryTech-App-Configs/podman/8016_inventree/inventree.env /home/inventree/.data/inventree/.env
    - Values to change:
      - INVENTREE_SITE_URL
      - TZ
      - INVENTREE_TIMEZONE
  - Create SystemD Quadlet
    - sudo -u inventree -i
    - cd /srv/CherryTech-App-Configs/podman/8016_inventree
    - podman quadlet install inventree-server.container inventree-worker.container inventree-db.container inventree-redis.container inventree-caddy.container inventree.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start inventree-pod.service
  - Full app initialization can take 5-10 minutes before the app will display
  - Configure Automatic Updates
    - systemctl --user enable --now podman-auto-update.timer

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/inventree/inventree
Wiki Link:
  - https://docs.inventree.org/en/stable/
Other Good Sources:
  - https://hub.docker.com/_/postgres
  - https://docs.inventree.org/en/stable/settings/error_codes/

Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
Most Postgres and MariaDB passwords can be long random passwords. Some systems however require specific charactors sets and requirements
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html


