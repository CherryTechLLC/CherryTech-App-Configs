Vaultwarden is a backend for the Bitwarden password manager without the limitations of the official self-hosted bitwarden server. It requires an email account to send details to users. Users will either request an account setup to thier email, or you can disable sign-up (recommended) and invite users manually from the admin panel.


DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp example.env .env
    - Values to change:
      - DOMAIN
      - TZ
      - SMTP Settings
        - https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration
  - Create a new dedicated user for this app
    - sudo useradd -m vaultwarden -F
    - sudo loginctl enable-linger vaultwarden
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
    - vaultwarden_admin_token  (https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
      -
    - vaultwarden_smtp_password
      - Create .vaultwarden_smtp_password.txt
      - Paste the password into the text file (By itself)
      - Login to that user (sudo -u vaultwarden -i)
      - Run podman secret create vaultwarden_smtp_password /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets/.vaultwarden_smtp_password.txt
    - Protect Secrets (MANDATORY)
      - sudo chown vaultwarden:vaultwarden /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
      - sudo chmod 700 /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
  - Networks and Volumes are automatically created
  - Start the App:
    - Login to app user (sudo -u vaultwarden -i)
    - podman-compose -f vaultwarden.yaml up -d
      - Use -d to detach the app from the terminal


Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/vaultwarden/server
Wiki Link:
  - https://github.com/dani-garcia/vaultwarden/wiki
Other Good Sources:
  - https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration
  - https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page



