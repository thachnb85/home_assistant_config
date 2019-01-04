Travis CI ![alt text](https://travis-ci.org/thachnb85/home_assistant_config.svg?branch=master "Travis CI Result")

# Hardware Overview

### Home Assistant
- Odroid XU4

### Zigbee Hub
- Xiaomi Aqara Hub for Zigbee sensor

### Sensors
- Xiaomi Door Sensors, Zigbee
- Xiaomi PIR Sensors

### IR/RF Blaster
- Broadlink RM Pro

### Cameras
- Raspberry with USB Camera
- MotionEye enabled with streaming option.

# Installation Guide

### 1. Install Ubuntu for Odroid

### 2. Install Home Assistant 
Guide link: https://www.home-assistant.io/docs/installation/raspberry-pi/

#### Prepare system
```
apt-get upgrade -y
apt-get install python3 python3-venv python3-pip
```
Install python3.6 on ubuntu 14.02
https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get

Symlink to make python3 point to 3.6 instead of 3.5
```
rm /usr/bin/python3
ln -s /usr/bin/python3.6 /usr/bin/python3```


#### Adding homeassistant user
```
useradd -rm homeassistant -G dialout
cd /srv
mkdir homeassistant
chown homeassistant:homeassistant -R /srv/homeassistant/
sudo -u homeassistant -H -s
```
#### Create virtual environment
```
cd /srv/homeassistant
python3 -m venv .
source bin/activate
python3 -m pip install wheel
pip3 install broadlink==0.9.0
pip3 install homeassistant
```

#### Run Home Assistant Server
```
hass
```

### Auto start homeassistant on boot
Check user we use on boot, here is root.

```nano /etc/systemd/system/home-assistant@root.service```

Adding content for files:
```
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/srv/homeassistant/bin/hass -c "/root/.homeassistant"

[Install]
WantedBy=multi-user.target
```

### Upgrade and Install python pkg if needed:
```
sudo systemctl stop home-assistant@homeassistant.service
sudo su -s /bin/bash homeassistant
source /srv/homeassistant/bin/activate
pip3 install --upgrade homeassistant
exit
sudo systemctl start home-assistant@homeassistant.service
```



