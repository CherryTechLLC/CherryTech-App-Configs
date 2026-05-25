Invoice Ninja is an excellent tool for creating and tracking invoices. Self-Hosting provides the Enterprise tier.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variables
    - sudo cp invoice-ninja.env .env
    - Values to change:
      - APP_URL
      - TZ
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/_invoice-ninja/secrets
    - Files:
      - Create .invoiceninja_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_db_root_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_admin_password.text
        - Paste the password into the text file (By itself)
      - App Key:
        - podman run --rm invoiceninja/invoiceninja-debian:latest php artisan key:generate --show
        - Create .invoiceninja_app_key.text
        - Paste the key into the text file including "base65:" at the start and "=" at the end.
  - Create a new dedicated user for this app
    - sudo useradd -m invoice-ninja -F
    - sudo loginctl enable-linger invoice-ninja
  - Protect Secrets (MANDATORY)
    - sudo chown -R invoiceninja:invoiceninja /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
    - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
  - Generate Secrets
    - Login to app user (sudo -u invoice-ninja -i
    - Run podman secret create invoiceninja_db_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/.invoiceninja_db_password
    - Run podman secret create invoiceninja_db_root_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/.invoiceninja_db_root_password
    - Run podman secret create invoiceninja_app_key /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/.invoiceninja_app_key
    - Run podman secret create invoiceninja_admin_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/.invoiceninja_admin_password
  - Prepare Bind Mounts
    - Login to app user (sudo -u invoice-ninja -i)
    - Build directory structure
      - mkdir /home/invoice-ninja/.data
      - mkdir /home/invoice-ninja/.data/invoice-ninja
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-public
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-storage
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-valkey
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-db
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-db-backups
      - cp /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/nginx /home/invoice-ninja/.data/invoice-ninja/ -r
    - chmod 770 -R /home/invoice-ninja/.data
  Create SystemD Quadlet
    - sudo -u invoiceninja -i
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - cd /srv/CherryTech-App-Configs/podman/8004_invoic-eninja
    - podman quadlet install -r --reload-systemd invoice-ninja-app.container invoice-ninja-db.container invoice-ninja-nginx.container invoice-ninja-redis.container invoice-ninja.pod

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





