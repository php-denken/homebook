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

```
apt-get install git-all
```

to authenticate towards github and so on

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Clone this repo...

### Basic

```
apt install -y curl ca-certificates gnupg htop ufw
```

Configure basic firewall

ufw allow ssh
ufw allow 80
ufw allow 443
ufw enable

### Docker

```
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

```
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable docker --now
usermod -aG docker $USER
```

## Setup services

All should run docker based so we create subfolders for that

```
mkdir -p ~/docker

cp /home/$GITFOLDER/homebook/docker/* ~/docker/
```

### Monitoring for tasmota

```
cd ~/docker/monitoring/mosquitto
docker compose up -d

docker network create monitoring

cd ~/docker/monitoring/influxdb
docker compose up -d
```

open
http://localhost:8086

Organisation: home
Bucket: power
Keep user pw and token

```
cd ~/docker/monitoring/telegraf
nano  telegraf.conf
```

Add token there

```
docker compose up -d
```

```
cd ~/docker/monitoring/grafana
docker compose up -d
```

http://localhost:3000

admin:admin

Data Source
Typ: InfluxDB
Query Language: Flux
URL: http://influxdb:8086
Organisation: home
Token: INFLUX_TOKEN
Default Bucket: power

### System monitoring

Create additional system bucket in influx

```
cd ~/docker/monitoring/telegraf-system
nano  telegraf.conf
```

Add token there

```
docker compose up -d
```


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

## Suspend disable

xfce ettings all off

xfce4-power-manager-settings

sudo nano /etc/systemd/logind.conf

IdleAction=ignore
IdleActionSec=0
HandleLidSwitch=ignore

sudo systemctl restart systemd-logind

## Minecraft

https://geysermc.org/download?project=floodgate

load 

Geyser‑Spigot.jar

Floodgate‑Spigot.jar


cp /home/FOO_BAR/Downloads/Geyser-Spigot.jar ./data/plugins/


load

Geyser‑Spigot.jar from
https://geysermc.org/download

Floodgate‑Spigot.jar from
https://geysermc.org/download?project=floodgate

and
https://www.curseforge.com/minecraft/bukkit-plugins/dynmap/files/7460127
https://www.spigotmc.org/resources/chunky.81534/
https://www.spigotmc.org/resources/spark.57242/

in the end the files

ls ./data/plugins/
Chunky-Bukkit-1.4.40.jar  Dynmap-3.8-spigot.jar  floodgate-spigot.jar  Geyser-Spigot.jar  spark-1.10.119-bukkit.jar

ufw allow 25565/tcp
ufw allow 19132/udp
ufw allow 8123/tcp

docker exec -it minecraft rcon-cli

op PLAYER_NAME
/dynmap pause
/dynmap resume

rm -rf ./data/world/
cp -r /home/FOO_BAR/Downloads/CITY/* ./data/world/
sudo chown -R 1000:1000 ./data/world
rm -rf ./data/world/players

cp -r ~/Downloads/<WORLD> ./data/world

# Pihole

I had to use native 53 ports
docker pot forwarding was not working for me

cd pihole

nano docker-compose.yml
#change PW

sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw allow 8083/tcp

sudo ufw status

docker compose up -d

Fritzbox

Internet → Zugangsdaten → DNS-Server

set homeserver ip as dns and alterntive dns

192.168.XXX.XXX

set no 1.1.1.1 or 9.9.9.9 as alternative dns so no way arround the pihole exists

Heimnetz → Netzwerk → Netzwerkeinstellungen → IP-Adressen → IPv4-Einstellungen

DHCP-Server aktivieren

#Homeassistent

cd homeassitent

docker compose up -d

Traedfri ikea gateway is connected via lan should set fixed ip and kan be integrated in the gui

Tasmota for energy measurement basically can alo be added with ip and shows its values
but so far i was not able to integrate it so that homeassistant undertands it as enrgy meter.

Next get a SONOFF Zigbee 3.0 & Thread Dongle...

