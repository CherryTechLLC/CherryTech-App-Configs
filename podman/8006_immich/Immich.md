Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/8006_immich/example.env /srv/CherryTech-App-Configs/podman/8006_immich/.env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m immich -F
    - sudo loginctl enable-linger immich
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Files:
      - Create .immich-db-password.txt
        - Paste the password into the text file (By itself)
        - Password must be all letters (no numbers or special charactors)
    - Protect Secrets (MANDATORY)
      - sudo chown -R immich:immich /srv/CherryTech-App-Configs/podman/8006_immich/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Generate Secrets
      - Login to app user (sudo -u immich -i)
      - Run podman secret create immich-db-password /srv/CherryTech-App-Configs/podman/8006_immich/secrets/.immich-db-password
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
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - cd /srv/CherryTech-App-Configs/podman/8006_immich
    - podman quadlet install -r --reload-systemd immich-app.container immich-db.container immich-machine-learning.container immich-redis.container immich.pod

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




