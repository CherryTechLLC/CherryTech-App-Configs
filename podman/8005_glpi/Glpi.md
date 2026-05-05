GLPI is a ITSM (IT Service Management) tool for asset tracking, ticketing, and more.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp glpi.env .env
    - Values to change:
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m glpi -F
    - sudo loginctl enable-linger glpi
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
    - Files:
      - Create .glpi_db_password.txt
        - Paste the password into the text file (By itself)
      - Create glpi_db_root_password.text
        - Paste the password into the text file (By itself)
      - Create .glpi_redis_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown -R glpi:glpi /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
      - sudo chmod -R 700 /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
    - Generate Secrets
      - Login to app user (sudo -u glpi -i
      - Run podman secret create glpi_db_password /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
      - Run podman secret create glpi_db_root_password /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
      - Run podman secret create glpi_redis_password /srv/CherryTech-App-Configs/podman/8005_glpi/secrets
  - Start the App:
    - Login to app user (sudo -u glpi -i)
    - podman-compose -f glpi.yaml up -d
      - Use -d to detach the app from the terminal

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
