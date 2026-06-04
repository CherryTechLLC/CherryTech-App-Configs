Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp activepieces.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m activepieces -F
    - sudo loginctl enable-linger activepieces
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets
    - Files: (Unless Otherwise Mentioned, Paste the password into the text file (By itself))
      - Create .activepieces-db-password.txt
      - Create .activepieces-redis-password.txt
      - Create .activepieces-api-key.text
      - Create .activepieces-encryption-key.txt
        - sudo openssl rand -hex 32
      - Create .activepieces-jwt-secret.txt
        - sudo openssl rand -hex 32
    - Protect Secrets (MANDATORY)
      - sudo chown -R activepieces:activepieces /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets
    - Generate Secrets
      - Login to app user (sudo -u activepieces -i
      - podman secret create activepieces-db-password /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets/.activepieces-db-password.txt
      - podman secret create activepieces-redis-password /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets/.activepieces-redis-password.txt
      - podman secret create activepieces-api-key /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets/.activepieces_api-key.txt
      - podman secret create activepieces-encryption-key /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets/.activepieces-encryption-key.txt
      - podman secret create activepieces-jwt-secrets /srv/CherryTech-App-Configs/podman/7001_activepieces/secrets/.activepieces-jwt-secret.txt
  - Create storage directories: (sudo -u immich -i)
    - mkdir /home/activepieces/.data/activepieces/activepieces-cache
    - mkdir /home/activepieces/.data/activepieces/activepieces-db
    - mkdir /home/activepieces/.data/activepieces/activepieces-db-backups
    - mkdir /home/activepieces/.data/activepieces/activepieces-valkey
  - Create SystemD Quadlet
    - sudo -u activepieces -i
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - cd /srv/CherryTech-App-Configs/podman/7001_activepieces
    - podman quadlet install -r --reload-systemd activepieces-app.container activepieces-worker.container activepieces-db.container activepieces-redis.container activepieces.pod

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  -
Other Good Sources:
  -


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html


