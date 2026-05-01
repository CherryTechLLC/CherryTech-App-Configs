DEPLOYMENT INSTRUCTIONS:
  - Create a new user for this app (useradd -m {{app-name}} -F)
  - Login to that user (sudo -u {{app name}} -i)
  - Set Domain and TZ env
  - Adjust resource limits for user count
  - Create Secrets (Per list at bottom of Yaml)
  - Networks and Volumes are automatically created
  - App Specific Requirements:
    - Create a mnt point on a NAS at /mnt/immich or have a large app disk
  - Run: podman-compose -f immich.yaml up
    - Use -d to detach the app from the terminal

Resource Limit Suggestions:
  - Use "podman stat" and check active use
  - Check online resources
  - Limits prevent memory leaks or other issue from crashing the whole server

Docker Hub Link:
  - N/A
Wiki Link:
  - https://docs.immich.app/install/docker-compose/
Other Good Sources:
  - https://docs.immich.app/install/environment-variables - Compose Options

Secrets:
  - Secrets prevent confidential data from being visible to other user accounts. Create each secret with a new .txt file (immich: nextcloud_db_password.txt) and past the password there.
  - IMMICH RULE: Please use only the characters `A-Za-z0-9`, without special characters or spaces (For DB password)


Then run:
  - podman secret create {{secret-name}} {{secret-name}}.txt

https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html








NOTES TO DEVELOPER:

  - Use official app images only!
  - Declare a source, default is docker.io/
  - Use :latest or decalare a version.

  - Check if you need to decalare TRUSTED_PROXIES for Reverse Proxy Config per app.

  - Use Redis when supported and recommended to improve performance.
  - Use PostGresSQL when supported and recommended as it is more feature rich and provides future options.


Health Check:
  - Below is a basic immich intended to see if an apps web server is active. This is what depends-on: condition: service_healthy requires to function.
  - DB and Redis need health checks, but the apps themselves don't always need the health check.

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
