These instructions walk through setting up Caddy with Hetzner DNS

In order for the domains you assign to applications to work you must create a DNS A record in the DNS server. I recommend something simple like *.apps.example.com and then in Caddy you assign your-app.app.example.com.


ON DEBIAN

sudo apt-get install caddy golang (Will delete, sets up service and user and everything)

sudo apt-mark hold caddy


CHECK FOR NEW CADDY VERSION BEFORE RUNNING THIS NEXT COMMAND

cd /tmp

wget https://github.com/caddyserver/xcaddy/releases/download/v0.4.7/xcaddy_0.4.7_linux_amd64.tar.gz

tar xvf xcaddy_0.4.7_linux_amd64.tar.gz xcaddy

sudo mv xcaddy /usr/bin

xcaddy build --with github.com/caddy-dns/hetzner/v2

sudo systemctl stop caddy

sudo cp /tmp/caddy /usr/bin/caddy

sudo systemctl stop caddy

caddy build-info

sudo chown caddy:caddy /etc/caddy/Caddyfile

sudo cp /srv/CherryTech-App-Configs/caddy/caddy.env.template /etc/caddy/caddy.env

sudo nano /etc/caddy/caddy.env

Create a hetzner API token and copy it to the HETZNER_API_TOKEN field in the .env file
Set the domain to the base domain of your applications, typicallt apps.example.com or just example.com

sudo chmod 600 /etc/caddy/caddy.env

sudo chown caddy:caddy /etc/caddy/caddy.env

sudo systemctl edit caddy

Add:
[Service]
EnvironmentFile=/etc/caddy/caddy.env

sudo cp /srv/CherryTech-App-Configs/caddy/Caddyfile /etc/caddy/Caddyfile

sudo systemctl daemon-reload

sudo systemctl enable --now caddy


Configure the firewall:
- sudo ufw allow 22/tcp
- sudo ufw allow 443/tcp
- sudo ufw default deny incoming
- sudo ufw default allow outgoing
- sudo ufw default deny routed
- sudo ufw enable
- sudo ufw status verbose
