Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Configure Environment Variable
    - sudo cp example.env .env
    - Values to change:
      - DOMAIN
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m EXAMPLE -F
    - sudo loginctl enable-linger EXAMPLE
  - Create secrets:
    - Create directory /srv/CherryTech-App-Configs/podman/{{PORT}}_EXAMPLE/secrets
    - Files:
      - Create .EXAMPLE_db_password.txt
        - Paste the password into the text file (By itself)
      - Create .EXAMPLE_redis_password.txt
        - Paste the password into the text file (By itself)
    - Protect Secrets (MANDATORY)
      - sudo chown vaultwarden:vaultwarden /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
      - sudo chmod 700 /srv/CherryTech-App-Configs/podman/8001_vaultwarden/secrets
    - Generate Secrets
      - Login to app user (sudo -u EXAMPLE -i
      - Run podman secret create EXAMPLE_db_password /srv/CherryTech-App-Configs/podman/{{PORT}}_EXAMPLE/secrets
      - Run podman secret create EXAMPLE_redis_password /srv/CherryTech-App-Configs/podman/{{PORT}}_EXAMPLE/secrets
  - Start the App:
    - Login to app user (sudo -u EXAMPLE -i)
    - podman-compose -f EXAMPLE.yaml up -d
      - Use -d to detach the app from the terminal

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  -
Wiki Link:
  -
Other Good Sources:
  -


Secrets:
Secrets ensure confidential information like database and admin passwords are only visible to superusers and the application's dedicated user.
https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html







NOTES TO DEVELOPER:

  - Use official app images only!
  - Declare a source, default is docker.io/
  - Use :latest or decalare a version.

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



LINKS:
  - Yaml Validator:  https://codebeautify.org/yaml-validator/cb4113054d-71c4-4759-b710-50da7fd1436a
  - Yaml Lint:  https://www.yamllint.com/
  - Dive: https://github.com/wagoodman/dive
