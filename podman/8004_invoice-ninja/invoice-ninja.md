Invoice Ninja is an excellent tool for creating and tracking invoices. Self-Hosting provides the Enterprise tier.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp invoice-ninja.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m invoice-ninja -F
    - sudo loginctl enable-linger invoice-ninja
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/_invoice-ninja/secrets
    - Files:
      - Create .invoice-ninja_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoice-ninja_db_root_password.txt
        - Paste the password into the text file (By itself)
      - Create .invoice-ninja_redis_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown vaultwarden:vaultwarden /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
      - sudo chmod 700 /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
    - Generate Secrets
      - Login to app user (sudo -u invoice-ninja -i
      - Run podman secret create invoice-ninja_db_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
      - Run podman secret create invoice-ninja_db_root_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
      - Run podman secret create invoice-ninja_redis_password /srv/CherryTech-App-Configs/podman/8004_invoice-ninja/secrets
  - Start the App:
    - Login to app user (sudo -u invoice-ninja -i)
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


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html





