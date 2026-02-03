# homebook
Using old hardware as homeserver

# OS

Debian
- Debian desktop env
- XFCE
- Standard system utilities
- SSH server
- NO X GNOME to save energy

## Additional software

All as root user...

su -

### Git

git

apt-get install git-all

to authenticate towards github and so on
ssh-keygen -t ed25519 -C "your_email@example.com"

Clone this repo...

### Basic

apt install -y curl ca-certificates gnupg htop ufw

Configure basic firewall

ufw allow ssh
ufw allow 80
ufw allow 443
ufw enable

### Docker


curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable docker --now
usermod -aG docker $USER

## Setup services

All should run docker based so we create subfolders for that

mkdir -p ~/docker

cp /home/$GITFOLDER/homebook/docker/* ~/docker/

### Monitoring for tasmota

cd ~/docker/monitoring/mosquitto
docker compose up -d

docker network create monitoring

cd ~/docker/monitoring/influxdb
docker compose up -d


open
http://localhost:8086

Organisation: home
Bucket: power
Keep user pw and token

cd ~/docker/monitoring/telegraf
nano  telegraf.conf

Add token there

docker compose up -d

cd ~/docker/monitoring/grafana
docker compose up -d

http://localhost:3000

admin:admin

Data Source
Typ: InfluxDB
Query Language: Flux
URL: http://influxdb:8086
Organisation: home
Token: INFLUX_TOKEN
Default Bucket: power


## Battery care for thinkpad

sudo apt update
sudo apt install tp-smapi-dkms acpi-call-dkms tlp tlp-rdw
sudo tlp-stat -b
should show battery

sudo nano /etc/systemd/system/battery-threshold.service

```
[Unit]
Description=Set ThinkPad Battery Charge Thresholds

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 40 > /sys/class/power_supply/BAT0/charge_start_threshold; echo 80 > /sys/class/power_supply/BAT0/charge_stop_threshold'

[Install]
WantedBy=multi-user.target
```

sudo systemctl enable battery-threshold.service
sudo systemctl start battery-threshold.service