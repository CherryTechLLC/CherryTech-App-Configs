ERPnext is a full business management suite for HR, Financing, CRM, and more. Note that it is a large and complex app.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp erpnext.env .env
    - Values to change:
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m erpnext -F
    - sudo loginctl enable-linger erpnext
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/8003_erpnext/secrets
    - Files:
      - Create .erpnext_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .erpnext_db_root_password.txt
        - Paste the password into the text file (By itself)
      - Create .erpnext_admin_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown -R vaultwarden:vaultwarden /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
      - sudo chmod -R 600 /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
    - Generate Secrets
      - Login to app user (sudo -u erpnext -i
      - Run podman secret create erpnext_db_password /srv/CherryTech-App-Configs/podman/{8003_erpnext/secrets/.erpnext_db_password.txt
      - Run podman secret create erpnext_db_root_password /srv/CherryTech-App-Configs/podman/8003_erpnext/secrets/.erpnext_db_root_password.txt
      - Run podman secret create erpnext_admin_password /srv/CherryTech-App-Configs/podman/{8003_erpnext/secrets/.erpnext_admin_password.txt
  - Start the App:
    - Login to app user (sudo -u erpnext -i)
    - podman-compose -f erpnext.yaml up -d
      - Use -d to detach the app from the terminal

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/frappe/erpnext
Wiki Link:
  - https://github.com/frappe/frappe_docker/blob/main/docs/getting-started.md
Other Good Sources:
  - https://github.com/frappe/frappe_docker


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
