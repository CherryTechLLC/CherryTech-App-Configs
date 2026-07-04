Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/8007_paperless/paperless.env .env
    - Values to change:
      - PAPERLESS_URL
      - TZ
      - PAPERLESS_ADMIN_MAIL
  - Create a new dedicated user for this app
    - sudo useradd -m paperless -F -u 8007
    - sudo loginctl enable-linger paperless
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8007_paperless/secrets
    - Files: (Paste text into file by itself)
      - nano .paperless-db-password.txt
      - nano .paperless-admin-password.txt
      - nano .paperless-secret-key.txt
        - openssl rand -base64 128
    - Protect Secrets (MANDATORY)
      - sudo chown -R paperless:paperless /srv/CherryTech-App-Configs/podman/8007_paperless/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8007_paperless/secrets
    - Generate Secrets
      - Login to app user (sudo -u paperless -i)
      - podman secret create paperless-db-password .paperless-db-password.txt
      - podman secret create paperless-admin-password .paperless-admin-password.txt
      - podman secret create paperless-secret-key .paperless-secret-key.txt
  - Create storage directories: (sudo -u paperless -i)
    - mkdir /home/paperless/.data/paperless/paperless-db
    - mkdir /home/paperless/.data/paperless/paperless-db-backups
    - mkdir /home/paperless/.data/paperless/paperless-valkey
    - mkdir /home/paperless/.data/paperless/paperless-data
    - mkdir /home/paperless/.data/paperless/paperless-export
    - mkdir /home/paperless/.data/paperless/paperless-consume
  - Create SystemD Quadlet
    - sudo -u paperless -i
    - cd /srv/CherryTech-App-Configs/podman/8007_paperless
    - podman quadlet install paperless-app.container paperless-db.container paperless-redis.container paperless.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start paperless-pod.service
  - Full app initialization can take 5-10 minutes before the app will display

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  - https://docs.paperless-ngx.com/setup/#installation_1
Other Good Sources:
  - https://hub.docker.com/_/postgres
  - https://docs.paperless-ngx.com/configuration/


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
  - Below is a basic paperless intended to see if an apps web server is active. This is what depends-on: condition: service_healthy requires to function.
  - Health checks can run constantly and be used to provide app status information.

healthcheck:
  test: ["CMD-SHELL", "curl -fsS http://localhost/ || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s
