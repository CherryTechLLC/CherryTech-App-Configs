Briefly explain the app and key notes here.

DEPLOYMENT INSTRUCTIONS:
  - Follow guide to clone repo from Github
  - Ensure Caddy Reverse Proxy is set up to display this app
  - Configure Environment Variable
    - sudo cp /srv/CherryTech-App-Configs/podman/8015_actual/actual.env .env
    - Values to change:
      - TZ
  - Create a new dedicated user for this app
    - sudo useradd -m actual -F -u 8015
    - sudo loginctl enable-linger actual
  - Create storage directories: (sudo -u actual -i)
    - mkdir /home/actual/.data/actual/actual-app
  - Create SystemD Quadlet
    - sudo -u actual -i
    - cd /srv/CherryTech-App-Configs/podman/8015_actual
    - podman quadlet install actual-app.container
    - export XDG_RUNTIME_DIR="/run/user/$UID" export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
    - systemctl --user daemon-reload
    - systemctl --user start actual-app.service
  - Full app initialization can take 5-10 minutes before the app will display

Resource Limits:
The resource limits provided are intended to prevent full server crashes should an app have a resource management issues. The provided values are for 5-10 users roughly, if you have more users you may have to increase the limits.
The CentOS cockpit application {{SERVER-IP}}:9090 shows you current usage of the app, if the usage is closing in on the limts (from more users) increase the limits.

Docker Hub Link:
  - https://hub.docker.com/r/actualbudget/actual-server
Wiki Link:
  - https://actualbudget.org/docs/
Other Good Sources:
  - https://github.com/actualbudget/actual/tree/master
  - https://github.com/actualbudget/actual/blob/master/packages/sync-server/docker-compose.yml
  - https://actualbudget.org/docs/config/reverse-proxies/
