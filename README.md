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
https://wiki.odroid.com/odroid-xu4/os_images/linux/start
Recommend kernel 4.14

### 2. Install Home Assistant 
Guide link: https://www.home-assistant.io/docs/installation/raspberry-pi/

#### Prepare system
```
apt-get upgrade -y
apt-get install build-essential libssl-dev libffi-dev python3 python3-venv python3-pip

```

Install python3.6 on ubuntu 14.02
```
sudo add-apt-repository ppa:jonathonf/python-3.6
sudo apt-get update
sudo apt-get install python3.6
sudo apt-get dist-upgrade
sudo apt-get install python-pip python3.6-dev
sudo pip install --upgrade virtualenv
```


#### Adding homeassistant user
```
sudo adduser --system homeassistant
sudo addgroup homeassistant
sudo usermod -G dialout -a homeassistant
sudo mkdir /srv/homeassistant

sudo chown homeassistant:homeassistant /srv/homeassistant
sudo su -s /bin/bash homeassistant

```
#### Create virtual environment
```
virtualenv -p python3.6 /srv/homeassistant
source /srv/homeassistant/bin/activate
pip3 install --upgrade homeassistant
pip3 install wheel
pip3 install broadlink==0.9.0
```

### Auto start homeassistant on boot
Check user we use on boot, here is root.

```nano /etc/systemd/system/home-assistant@homeassistant.service```

Adding content for files:
```
[Unit]
Description=Home Assistant
After=network.target time-sync.target
Requires=time-sync.target

[Service]
Type=simple
User=%i
ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target
```

Then start it:
```
sudo systemctl --system daemon-reload
sudo systemctl start home-assistant@homeassistant.service
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
### Port forwarding and SSL
https://www.youtube.com/watch?v=BIvQ8x_iTNE

### Problem?
In case of problems, you will be able to review the logs through journalctl:
```
sudo journalctl -u homeassistant -f
```



