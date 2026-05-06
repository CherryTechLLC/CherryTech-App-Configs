Vaultwarden is a backend for the Bitwarden password manager without the limitations of the official self-hosted bitwarden server. It requires an email account to send details to users. Users will either request an account setup to thier email, or you can disable sign-up (recommended) and invite users manually from the admin panel.


DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp example.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m vaultwarden -F
    - sudo loginctl enable-linger vaultwarden
    - export XDG_RUNTIME_DIR="/run/user/$UID"
      export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
  - Create Networks and Volumes: (sudo -u vaultwarden -i)
    - mkdir /home/vaultwarden/.config/vaultwarden/
    - mkdir /home/vaultwarden/.data/vaultwarden/vaultwarden-app
    - podman network create vaultwarden-net
  - Create SystemD
  - Log in to vault.apps.example.com/admin with the token CHANGE-ME
  - Create secure vaultwarden_admin_token  (https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
      - sudo dnf install epel-release -y
      - sudo dnf install argon2 -y
      - echo -n 'EXAMPLE-PASSWORD' | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4
      - Copy the ENTIRE string including $argon2id$ into the "Admin token/Argon2 PHC" section of the general settings
      - Select save on the bottom
      - log out (top right)
      - Enter the password you used to generate the Argon2 string
  - Configure SMTP: (Simple Mail Transer Protocol, required.)
    - Enter the hostname for your email server (mail.your-server.de for Hetzner)
    - Set the email address information will come from (vaulwarden.example.com)
    - From Name is up to you.
    - Username is generally the email address. (vaultwarden.example.com)
    - Password is the SMTP account password.


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



