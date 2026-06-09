Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/8006_immich/example.env /srv/CherryTech-App-Configs/podman/8006_immich/.env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m immich -F -u 8006
    - sudo loginctl enable-linger immich
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Files: (Paste text into file by itself)
      - Create .immich-db-password.txt
        - Password must be all letters (no numbers or special charactors)
    - Protect Secrets (MANDATORY)
      - sudo chown -R immich:immich /srv/CherryTech-App-Configs/podman/8006_immich/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Generate Secrets
      - Login to app user (sudo -u immich -i)
      - podman secret create immich-db-password .immich-db-password.txt
  - Create storage directories: (sudo -u immich -i)
    - mkdir /home/immich/.data/immich/immich-db
    - mkdir /home/immich/.data/immich/immich-db-backups
    - mkdir /home/immich/.data/immich/immich-model-cache
    - mkdir /home/immich/.data/immich/immich-valkey
    - mkdir /home/immich/.data/immich/immich-uploads
  - OPTIONAL: Set up NFS on NAS
    - Grant Root user all priviledges on share (on NAS)
    - On NFS share set CentOS server IP as allowed host and Mapall to Root. (All access from that IP is given the root account tag)
    - sudo dnf install nfs-utils -y
    - sudo nano /etc/fstab
      - {{NAS IP}}:/mnt/{{share-name}}/Immich /home/immich/.data/immich/immich-uploads nfs defaults,_netdev 0 0
    - sudo systemctl daemon-reload
    - sudo mount -a
  - Create SystemD Quadlet
    - sudo -u immich -i
    - cd /srv/CherryTech-App-Configs/podman/8006_immich
    - podman quadlet install immich-app.container immich-db.container immich-machine-learning.container immich-redis.container immich.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start immich-pod.service
  - Full app initialization can take 5-10 minutes before the app will display

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - N/A
Wiki Link:
  - https://docs.immich.app/install/docker-compose/
Other Good Sources:
  - https://docs.immich.app/install/environment-variables - Compose Options
  - https://github.com/jbtrystram/immich-podman-systemd/tree/main

Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html




