Netbox is a utility for tracking networks and server rack equipment for infratructure documentation

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Create a new dedicated user for this app
    - sudo useradd -m netbox -F -u 8008
    - sudo loginctl enable-linger netbox
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8008_netbox/secrets
    - Files: (Paste text into file by itself)
      - nano .netbox-db-password.txt
        - sudo openssl rand -base64 64
      - nano .netbox-pepper1-api.txt
        - sudo openssl rand -base64 64
      - nano .netbox-secret-key.txt
        - sudo openssl rand -base64 64
    - Protect Secrets (MANDATORY)
      - sudo chown -R netbox:netbox /srv/CherryTech-App-Configs/podman/8008_netbox/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8008_netbox/secrets
    - Generate Secrets
      - Login to app user (sudo -u netbox -i)
      - podman secret create netbox-db-password .netbox-db-password.txt
      - podman secret create netbox-pepper1-api .netbox-pepper1-api.txt
      - podman secret create netbox-secret-key .netbox-secret-key.txt
  - Create storage directories: (sudo -u netbox -i)
    - mkdir /home/netbox/.data/netbox/netbox-db
    - mkdir /home/netbox/.data/netbox/netbox-db-backups
    - mkdir /home/netbox/.data/netbox/netbox-valkey
    - mkdir /home/netbox/.data/netbox/netbox-valkey-cache
    - mkdir /home/netbox/.data/netbox/netbox-media
    - mkdir /home/netbox/.data/netbox/netbox-reports
    - mkdir /home/netbox/.data/netbox/netbox-scripts
  - Configure Environment Variable (sudo -u netbox -i)
  - Configure Environment Variable
    - cp /srv/CherryTech-App-Configs/podman/8008_netbox/netbox.env /home/netbox/.data/netbox/.env
    - Values to change:
      - ALLOWED_HOSTS
      - TZ
  - Create netbox configuration file (sudo -u netbox -i)
  - Create netbox configuration file
    - cp /srv/CherryTech-App-Configs/podman/8008_netbox/config /home/netbox/.data/netbox/.config
  - Create SystemD Quadlet
    - sudo -u netbox -i
    - cd /srv/CherryTech-App-Configs/podman/8008_netbox
    - podman quadlet install netbox-app.container netbox-worker.container netbox-db.container netbox-redis.container netbox-redis-cache.container netbox.network
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start netbox-pod.service
  - Full app initialization can take 5-10 minutes before the app will display
  - Configure Automatic Updates
    - systemctl --user enable --now podman-auto-update.timer

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/netboxcommunity/netbox/
Wiki Link:
  - https://github.com/netbox-community/netbox-docker/wiki
Other Good Sources:
  - https://github.com/netbox-community/netbox-docker/blob/release/env/netbox.env
  - https://hub.docker.com/_/postgres

Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
Most Postgres and MariaDB passwords can be long random passwords. Some systems however require specific charactors sets and requirements
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html



