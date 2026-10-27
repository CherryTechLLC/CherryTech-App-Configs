Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Create a new dedicated user for this app
    - sudo useradd -m inventree -F -u 8016
    - sudo loginctl enable-linger inventree
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
    - Files: (Paste text into file by itself)
      - sudo nano .inventree-db-password.txt
        - sudo openssl rand -base64 64
      - sudo nano .inventree-secret-key.txt
        - sudo openssl rand -base64 128
    - Protect Secrets (MANDATORY)
      - sudo chown -R inventree:inventree /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8016_inventree/secrets
    - Generate Secrets
      - Login to app user (sudo -u inventree -i)
      - podman secret create inventree-db-password .inventree-db-password.txt
      - podman secret create inventree-secret-key .inventree-secret-key.txt
  - Create storage directories: (sudo -u inventree -i)
    - mkdir /home/inventree/.data/inventree/inventree-db
    - mkdir /home/inventree/.data/inventree/inventree-db-backups
    - mkdir /home/inventree/.data/inventree/inventree-valkey
    - mkdir /home/inventree/.data/inventree/inventree-app
    - mkdir /home/inventree/.data/inventree/inventree-config
    - cp /srv/CherryTech-App-Configs/podman/8016_inventree/Caddyfile.internal /home/inventree/.data/inventree/Caddyfile
  - Configure Environment Variable (sudo -u inventree -i)
    - cp /srv/CherryTech-App-Configs/podman/8016_inventree/inventree.env /home/inventree/.data/inventree/.env
    - Values to change:
      - INVENTREE_SITE_URL
      - TZ
  - Create SystemD Quadlet
    - sudo -u inventree -i
    - cd /srv/CherryTech-App-Configs/podman/8016_inventree
    - podman quadlet install inventree-server.container inventree-worker.container inventree-db.container inventree-redis.container inventree.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start inventree-pod.service
  - Full app initialization can take 5-10 minutes before the app will display
  - Configure Automatic Updates
    - systemctl --user enable --now podman-auto-update.timer

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  -
Other Good Sources:
  - https://hub.docker.com/_/mariadb
  - https://hub.docker.com/_/postgres

Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
Most Postgres and MariaDB passwords can be long random passwords. Some systems however require specific charactors sets and requirements
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html




NOTES TO DEVELOPER:

  - Use official app images only!
  - Declare a source, default is docker.io/
  - Use :latest or decalare a version.
  - Add a Caddy Entry so apps get exposed
  - Stuff you may not need has been added by default becuase deleting is easier than adding
  - In the .pod file UserNS=keep-id can cause permission but otherwise is important

  - Check if you need to decalare TRUSTED_PROXIES for Reverse Proxy Config per app.
  - SMTP (Simple mail transfer protocol) is used to allow applications to send emails. The standard port is 587 on modern servers with starttls for security, each app has different environment variables to set.

  - Use Redis when supported and recommended to improve performance.
    - Redis is closed source now, but Valkey is equivilent and doesnt have the 50mb limit. That image is used on the redis container.
  - Use PostGresSQL when supported and recommended as it is more feature rich and provides future options.


Health Check:
  - Below is a basic inventree intended to see if an apps web server is active. This is what depends-on: condition: service_healthy requires to function.
  - Health checks can run constantly and be used to provide app status information.

healthcheck:
  test: ["CMD-SHELL", "curl -fsS http://localhost/ || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s
