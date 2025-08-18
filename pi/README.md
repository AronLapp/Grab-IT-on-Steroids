# RaspberryPi 3 1GB Setup

## Basis

Ubuntu Server 22.04.5 LTS (64-bit) "preinstalled image“

## WLAN via Netplan eingerichten

Datei ```/etc/netplan/01-netcfg.yaml```

```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: true
      optional: true
      access-points:
        "WiFi-SSID":
          password: "WiFi-Password"

```

### Neue Netzwerkkonfiguration anwenden

```bash
sudo netplan generate && sudo netplan apply
```

### Optional für schnelleren Bootvorgang

```bash
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
```

## ROS2 Humble installieren

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y locales software-properties-common curl gnupg lsb-release
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo add-apt-repository universe -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=arm64 signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/ros2.list

sudo apt update
sudo apt install -y ros-humble-ros-base ros-humble-demo-nodes-cpp ros-humble-demo-nodes-py
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## ZRAM expansion (für schwächere PIs mit wenig RAM)

Skript unter ```/usr/local/bin/zram-1g.sh``` speichern

```bash
#!/bin/bash
modprobe zram
echo $((1024*1024*1024)) | tee /sys/block/zram0/disksize
mkswap /dev/zram0
swapon /dev/zram0
```

### Service ```/etc/systemd/system/zram-1g.service```

```service
[Unit]
Description=ZRAM swap (1GiB)
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/zram-1g.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

```bash
sudo chmod +x /usr/local/bin/zram-1g.sh
sudo systemctl daemon-reload
sudo systemctl enable --now zram-1g.service
```

### Alte Swap-Datei deaktivieren (SDCard schonen)

```bash
sudo dphys-swapfile swapoff
sudo systemctl disable dphys-swapfile
sudo rm /var/swap
```

### Test

```bash
free -h
swapon --show
```
