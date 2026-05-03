Nextcloud is a Google Drive and M365 alternative that is open source, fully self hosted, and has a huge library of apps and plugins available to expand its capabilities.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp example.env .env
    - Values to change:
      - DOMAIN
      - TZ
      - SMTP Settings
  - Create a new dedicated user for this app
    - sudo useradd -m nextcloud -F
    - sudo loginctl enable-linger nextcloud
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
    - Files:
      - Create .nextcloud_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .nextcloud_redis_password.txt
        - Paste the password into the text file (By itself)
      - Create .nextcloud_smtp_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown vaultwarden:vaultwarden /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
      - sudo chmod 700 /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
    - Generate Secrets
      - Login to app user (sudo -u nextcloud -i
      - Run podman secret create vaultwarden_db_password /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets/.nextcloud_db_password.txt
      - Run podman secret create vaultwarden_redis_password /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets/.nextcloud_redis_password.txt
      - Run podman secret create vaultwarden_smtp_password /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets/.nextcloud_smtp_password.txt
  - Networks and Volumes are automatically created
  - Start the App:
    - Login to that user (sudo -u nextcloud -i)
    - podman-compose -f nextcloud.yaml up -d
      - Use -d to detach the app from the terminal


Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/_/nextcloud/
Wiki Link:
  - https://github.com/nextcloud/server/wiki
Other Good Sources:
  -


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
