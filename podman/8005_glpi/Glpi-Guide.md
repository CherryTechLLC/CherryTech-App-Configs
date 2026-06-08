GLPI is a ITSM (IT Service Management) tool for asset tracking, ticketing, and more.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variables
    - sudo cp glpi.env .env
    - Values to change:
      - TZ
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
    - Files: (Paste text into file by itself)
      - nano .glpi-db-password.txt
      - nano .glpi-db-root_password.txt
  - Create a new dedicated user for this app
    - sudo useradd -m glpi -F -u 8005
    - sudo loginctl enable-linger glpi
  - Protect Secrets (MANDATORY)
    - sudo chown -R glpi:glpi /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
    - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
  - Generate Secrets
    - Login to app user (sudo -u glpi -i
    - podman secret create glpi-db-password /srv/CherryTech-App-Configs/podman/8005_glpi/secrets/.glpi-db-password.txt
    - podman secret create glpi-db-root-password /srv/CherryTech-App-Configs/podman/8005_glpi/secrets/.glpi-db-root-password.txt
  - Prepare Bind Mounts
    - Login to app user (sudo -u glpi -i)
    - Build directory structure
      - mkdir /home/glpi/.data/glpi/glpi-app
      - mkdir /home/glpi/.data/glpi/glpi-marketplace
      - mkdir /home/glpi/.data/glpi/glpi-valkey
      - mkdir /home/glpi/.data/glpi/glpi-db
      - mkdir /home/glpi/.data/glpi/glpi-db-backups
  - Create SystemD Quadlet
    - sudo -u glpi -i
    - cd /srv/CherryTech-App-Configs/podman/8005_glpi
    - podman quadlet install glpi-app.container glpi-db.container glpi-redis.container glpi.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start glpi-pod.service
  - Full system intialization can take 5-10 minutes before the app will display
  - Login and configure
    - App url
    - Redis URL

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/glpi/glpi
Wiki Link:
  -
Other Good Sources:
  - https://github.com/glpi-project/glpi


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
