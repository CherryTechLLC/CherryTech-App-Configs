Invoice Ninja is an excellent tool for creating and tracking invoices. Self-Hosting provides the Enterprise tier.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variables
    - sudo cp invoiceninja.env .env
    - Values to change:
      - APP_URL
      - TZ
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/_invoiceninja/secrets
    - Files:
      - Create .invoiceninja_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_db_root_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_admin_password.text
        - Paste the password into the text file (By itself)
      - App Key:
        - sudo podman run --rm invoiceninja/invoiceninja-debian:latest php artisan key:generate --show
        - Create .invoiceninja_app_key.text
        - Paste the key into the text file including "base65:" at the start and "=" at the end.
  - Create a new dedicated user for this app
    - sudo useradd -m invoiceninja -F
    - sudo loginctl enable-linger invoiceninja
  - Protect Secrets (MANDATORY)
    - sudo chown -R invoiceninja:invoiceninja /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets
    - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets
  - Generate Secrets
    - Login to app user (sudo -u invoiceninja -i
    - podman secret create invoiceninja-db-password /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets/.invoiceninja-db-password.txt
    - podman secret create invoiceninja_db-root-password /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets/.invoiceninja-db-root-password.txt
    - podman secret create invoiceninja-app-key /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets/.invoiceninja-app_key.txt
    - podman secret create invoiceninja-admin-password /srv/CherryTech-App-Configs/podman/8004_invoiceninja/secrets/.invoiceninja-admin-password.txt
  - Prepare Bind Mounts
    - Login to app user (sudo -u invoiceninja -i)
    - Build directory structure
      - mkdir /home/invoiceninja/.data/invoiceninja/invoiceninja-public
      - mkdir /home/invoiceninja/.data/invoiceninja/invoiceninja-storage
      - mkdir /home/invoiceninja/.data/invoiceninja/invoiceninja-valkey
      - mkdir /home/invoiceninja/.data/invoiceninja/invoiceninja-db
      - mkdir /home/invoiceninja/.data/invoiceninja/invoiceninja-db-backups
      - cp /srv/CherryTech-App-Configs/podman/8004_invoiceninja/nginx /home/invoiceninja/.data/invoiceninja/ -r
    - chmod 770 -R /home/invoiceninja/.data
  Create SystemD Quadlet
    - sudo -u invoiceninja -i
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - cd /srv/CherryTech-App-Configs/podman/8004_invoiceninja
    - podman quadlet install -r --reload-systemd invoiceninja-app.container invoiceninja-db.container invoiceninja-nginx.container invoiceninja-redis.container invoiceninja.pod

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/invoiceninja/invoiceninja-debian
Wiki Link:
  -
Other Good Sources:
  - https://github.com/invoiceninja/invoiceninja/blob/master/.env.example
  - https://invoiceninja.github.io/docs/self-host/env-variables


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html





