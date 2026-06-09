Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/example.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m EXAMPLE -F -u 0000
    - sudo loginctl enable-linger EXAMPLE
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
    - Files: (Paste text into file by itself)
      - nano .EXAMPLE-db-password.txt
      - nano .EXAMPLE-db-root-password.text
    - Protect Secrets (MANDATORY)
      - sudo chown -R EXAMPLE:EXAMPLE /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/0000_EXAMPLE/secrets
    - Generate Secrets
      - Login to app user (sudo -u EXAMPLE -i)
      - podman secret create EXAMPLE-db-password .EXAMPLE-db-password.txt
      - podman secret create EXAMPLE-db-root-password .EXAMPLE-db-root-password.txt
  - Create storage directories: (sudo -u EXAMPLE -i)
    - mkdir /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-db
    - mkdir /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-db-backups
    - mkdir /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-valkey
    - mkdir /home/EXAMPLE/.data/EXAMPLE/EXAMPLE-app
  - Create SystemD Quadlet
    - sudo -u EXAMPLE -i
    - cd /srv/CherryTech-App-Configs/podman/0000_EXAMPLE
    - podman quadlet install EXAMPLE-app.container EXAMPLE-db.container EXAMPLE-redis.container EXAMPLE.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start EXAMPLE-pod.service
  - Full app initialization can take 5-10 minutes before the app will display

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

  - Check if you need to decalare TRUSTED_PROXIES for Reverse Proxy Config per app.
  - SMTP (Simple mail transfer protocol) is used to allow applications to send emails. The standard port is 587 on modern servers with starttls for security, each app has different environment variables to set.

  - Use Redis when supported and recommended to improve performance.
    - Redis is closed source now, but Valkey is equivilent and doesnt have the 50mb limit. That image is used on the redis container.
  - Use PostGresSQL when supported and recommended as it is more feature rich and provides future options.


Health Check:
  - Below is a basic example intended to see if an apps web server is active. This is what depends-on: condition: service_healthy requires to function.
  - Health checks can run constantly and be used to provide app status information.

healthcheck:
  test: ["CMD-SHELL", "curl -fsS http://localhost/ || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s
