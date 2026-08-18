Authentik is a tool designed to provide strong user account control (UAC) for internally hosted applications.

DEPLOYMENT INSTRUCTIONS:
  - Create a fresh installation of Ubuntu Server
    - Ensure you have a very strong password
    - Update system
      - sudo apt update
      - sudo apt dist-upgrade
      - sudo apt autoremove
  - Install required packages
    - sudo apt-get install qemu-guest-agent git
      - Skip qemu-guest-agent if not using proxmox
  - Set up automatic updates
    - 
  - Follow Caddy set up guide
  - Configure Environment Variables
    - sudo cp /srv/CherryTech-App-Configs/podman/6002_authentik/authentik.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m authentik -F -u 6002
    - sudo loginctl enable-linger authentik
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
    - Files: (Paste text into file by itself)
      - nano .authentik-db-password.txt
      - nano .authentik-secret-key.txt
    - Protect Secrets (MANDATORY)
      - sudo chown -R authentik:authentik /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/6002_authentik/secrets
    - Generate Secrets
      - Login to app user (sudo -u authentik -i)
      - podman secret create authentik-db-password .authentik-db-password.txt
      - podman secret create authentik-secret-key .authentik-secret-key.txt
  - Create storage directories: (sudo -u authentik -i)
    - mkdir /home/authentik/.data/authentik/authentik-db
    - mkdir /home/authentik/.data/authentik/authentik-db-backups
    - mkdir /home/authentik/.data/authentik/authentik-app
    - mkdir /home/authentik/.data/authentik/authentik-custom-templates
    - mkdir /home/authentik/.data/authentik/authentik-certs
  - Create SystemD Quadlet
    - sudo -u authentik -i
    - cd /srv/CherryTech-App-Configs/podman/6002_authentik
    - podman quadlet install authentik-app.container authentik-db.container authentik-redis.container authentik.pod
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start authentik-pod.service
  - Full app initialization can take 5-10 minutes before the app will display

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  - https://docs.goauthentik.io/
Other Good Sources:
  - https://github.com/goauthentik/authentik
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
  - Below is a basic authentik intended to see if an apps web server is active. This is what depends-on: condition: service_healthy requires to function.
  - Health checks can run constantly and be used to provide app status information.

healthcheck:
  test: ["CMD-SHELL", "curl -fsS http://localhost/ || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s
