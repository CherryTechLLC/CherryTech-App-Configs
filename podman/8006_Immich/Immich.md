Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp immich.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m immich -F
    - sudo loginctl enable-linger immich
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Files:
      - Create .immich_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .immich_valkey_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown -R immich:immich /srv/CherryTech-App-Configs/podman/8006_immich/secrets
      - sudo chmod -R 600 /srv/CherryTech-App-Configs/podman/8006_immich/secrets
    - Generate Secrets
      - Login to app user (sudo -u immich -i
      - Run podman secret create immich_db_password /srv/CherryTech-App-Configs/podman/8006_immich/secrets
      - Run podman secret create immich_valkey_password /srv/CherryTech-App-Configs/podman/8006_immich/secrets
  - Start the App:
    - Login to app user (sudo -u immich -i)
    - podman-compose -f immich.yaml up -d
      - Use -d to detach the app from the terminal

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - N/A
Wiki Link:
  - https://docs.immich.app/install/docker-compose/
Other Good Sources:
  - https://docs.immich.app/install/environment-variables - Compose Options


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html




