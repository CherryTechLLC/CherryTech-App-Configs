Invoice Ninja is an excellent tool for creating and tracking invoices. Self-Hosting provides the Enterprise tier.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variables
    - sudo cp invoice-ninja.env .env
    - Values to change: (CHANGE PERMISSIONS ON .env))
      - DOMAIN
      - TZ
      - Admin Email (Leave as admin@admin.com)
      - Admin Password (Keep Complex, It is Hard Coded)
      - DB, and Valkey(Redis) Password (Use the same values as the secrets later) (CHANGE PERMISSIONS ON .env)
      - App Key:
        - podman run --rm invoiceninja/invoiceninja-debian:latest php artisan key:generate --show
        - Paste the entire base64 string into Environment file including "base65:" at the start and "=" at the end.
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/_invoice-ninja/secrets
    - Files:
      - Create .invoiceninja_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_db_root_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoiceninja_valkey_password.text
        - Paste the password into the text file (By itself)
  - Create a new dedicated user for this app
    - sudo useradd -m invoice-ninja -F
    - sudo loginctl enable-linger invoice-ninja
  - Protect Secrets (MANDATORY)
    - sudo chown -R invoiceninja:invoiceninja /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
    - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
  - Generate Secrets
    - Login to app user (sudo -u invoice-ninja -i
    - Run podman secret create invoiceninja_db_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/invoiceninja_db_password
    - Run podman secret create invoiceninja_db_root_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/invoiceninja_db_root_password
    - Run podman secret create invoiceninja_valkey_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/invoiceninja_valkey_password
    - Run podman secret create invoiceninja_app_key /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets/invoiceninja_app_key
  - Prepare Bind Mounts
    - Login to app user (sudo -u invoice-ninja -i)
    - Build directory structure
      - mkdir /home/invoice-ninja/.data
      - mkdir /home/invoice-ninja/.data/invoice-ninja
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-public
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-storage
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-valkey
      - mkdir /home/invoice-ninja/.data/invoice-ninja/invoiceninja-db
      - cp /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/nginx /home/invoice-ninja/.data/invoice-ninja/ -r
    - chmod 770 -R /home/invoice-ninja/.data
  - Start the App:
    - Login to app user (sudo -u invoice-ninja -i)
    - cd /srv/CherryTech-App-Configs/podman/8004_invoice-ninja
    - podman-compose -f invoice-ninja.yaml up -d
      - Use -d to detach the app from the terminal

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





