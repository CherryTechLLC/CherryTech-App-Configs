Nextcloud is a Google Drive and M365 alternative that is open source, fully self hosted, and has a huge library of apps and plugins available to expand its capabilities.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/8002_nextcloud/nextcloud.env .env
    - Values to change:
      - DOMAIN
      - TZ
      - NEXTCLOUD_TRUSTED_DOMAINS
      - OVERWRITEHOST
      - OVERWRITECLIURL
  - Create a new dedicated user for this app
    - sudo useradd -m nextcloud -F -u 8002
    - sudo loginctl enable-linger nextcloud
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8002_nextcloud/secrets
    - Files: (Paste text into file by itself)
      - nano .nextcloud-db-password.txt
    - Protect Secrets (MANDATORY)
      - sudo chown -R nextcloud:nextcloud /srv/CherryTech-App-Configs/podman/8002_nextcloud/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8002_nextcloud/secrets
    - Generate Secrets
      - Login to app user (sudo -u nextcloud -i)
      - podman secret create nextcloud-db-password .nextcloud-db-password.txt
  - Create storage directories: (sudo -u nextcloud -i)
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-db
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-db-backups
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-valkey
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-app
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-custom_apps
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-config
    - mkdir /home/nextcloud/.data/nextcloud/nextcloud-data
  - Create SystemD Quadlet
    - sudo -u nextcloud -i
    - cd /srv/CherryTech-App-Configs/podman/8002_nextcloud
    - podman quadlet install nextcloud-app.container nextcloud-db.container nextcloud-redis.container nextcloud.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start nextcloud-pod.service
  - Full app initialization can take 5-10 minutes before the app will display

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/_/nextcloud/
Wiki Link:
  - https://github.com/nextcloud/server/wiki
Other Good Sources:
  - Docker Health Check: https://github.com/nextcloud/docker/issues/676


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
