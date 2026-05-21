These instructions walk through setting up Caddy with Hetzner DNS

sudo dnf install go

sudo dnf copr enable @caddy/caddy

sudo dnf install caddy (Will delete, sets up service and user and everything)

sudo nano /etc/dnf/dnf.conf

Add the line:
exclude=caddy

CHECK FOR NEW CADDY VERSION BEFORE RUNNING THIS NEXT COMMAND

cd /tmp

wget https://github.com/caddyserver/xcaddy/releases/download/v0.4.5/xcaddy_0.4.5_linux_amd64.tar.gz

tar xvf xcaddy_0.4.5_linux_amd64.tar.gz xcaddy

sudo mv xcaddy /usr/bin

xcaddy build --with github.com/caddy-dns/hetzner/v2

sudo cp /tmp/caddy /usr/bin/caddy

caddy build-info

sudo chown caddy:caddy /etc/caddy/Caddyfile

sudo chmod 600 /etc/caddy/caddy.env

sudo chown caddy:caddy /etc/caddy/caddy.env

sudo systemctl edit caddy

Add:

[Service]
EnvironmentFile=/etc/caddy/caddy.env



sudo setsebool -P httpd_can_network_connect 1

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload

sudo systemctl daemon-reexec

sudo systemctl daemon-reload

sudo systemctl enable --now caddy
