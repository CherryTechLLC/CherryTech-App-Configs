DEPLOYMENT INSTRUCTIONS:
  - Create a new user for this app (useradd -m {{app-name}} -F)
  - Login to that user (sudo -u {{app name}} -i)
  - Set Domain and TZ env
  - Adjust resource limits for user count
  - Create Secrets (Per list at bottom of Yaml)
  - Networks and Volumes are automatically created
  - App Specific Requirements:
    - Admin Page: https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page
  - Run: podman-compose -f EXAMPLE.yaml up
    - Use -d to detach the app from the terminal

Resource Limit Suggestions:
  - Use "podman stat" and check active use
  - Check online resources
  - Limits prevent memory leaks or other issue from crashing the whole server

Docker Hub Link:
  - https://hub.docker.com/r/vaultwarden/server
Wiki Link:
  - https://github.com/dani-garcia/vaultwarden/wiki
Other Good Sources:
  -

Secrets:
  - Secrets prevent confidential data from being visible to other user accounts. Create each secret with a new .txt file (example: nextcloud_db_password.txt) and past the password there.

Then run:
  - podman secret create {{secret-name}} {{secret-name}}.txt

https://docs.podman.io/en/stable/markdown/podman-secret-create.1.html
