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

```nano /etc/systemd/system/home-assistant.service```

Adding content for files:
```
[Unit]
Description=Home Assistant
After=network.target time-sync.target
Requires=time-sync.target

[Service]
Type=simple
User=root
ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target
```

Try to run homeassistant first:
```
/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant
```

Then start it:
```
sudo systemctl --system daemon-reload
sudo systemctl enable home-assistant.service
sudo systemctl start home-assistant.service
```

### Upgrade and Install python pkg if needed:
```
sudo systemctl stop home-assistant.service
sudo su -s /bin/bash homeassistant
source /srv/homeassistant/bin/activate
pip3 install --upgrade homeassistant
exit
sudo systemctl start home-assistant.service
```
### Port forwarding and SSL
https://www.youtube.com/watch?v=BIvQ8x_iTNE

### Problem?
In case of problems, you will be able to review the logs through journalctl:
```
sudo journalctl -u homeassistant -f
```
# Setup zigbee2mqtt.io
### Hardware
- Grab a CC2531 usb stick
- There are 2 kinds of firmwares: Coordinator (allows pairing), and other is Router (just routing).
- https://www.zigbee2mqtt.io/getting_started/flashing_the_cc2531.html

### Install zigbee2mqtt and mosquitto broker
- https://www.zigbee2mqtt.io/getting_started/running_zigbee2mqtt.html
- Setup and run as service.
- Install broker: https://mosquitto.org
```
apt-get install mosquitto
```
- Test the broker:
```
apt install mosquitto-clients

mosquitto_sub -h 127.0.0.1 -t "test" &
mosquitto_pub -h 127.0.0.1 -t "test" -m "Hello World" 
```
-> "Hello World" should show up in the terminal
 
### Config with HA

1- Add user in HA, using web interface adding user: mqtt, password: mqtt

2- Edit the config of zigbee2mqtt
```
nano /opt/zigbee2mqtt/data/configuration.yaml
```
Then edit:
```
# Home Assistant integration (MQTT discovery)
homeassistant: true

# allow new devices to join
permit_join: true

# MQTT settings
mqtt:
  # MQTT base topic for zigbee2mqtt MQTT messages
  base_topic: zigbee2mqtt
  # MQTT server URL
  server: 'mqtt://localhost'
  # MQTT server authentication, uncomment if required:
  user: mqtt
  password: mqtt

# Serial settings
serial:
  # Location of CC2531 USB sniffer
  port: /dev/ttyACM0
```

- Check if setup is properly or not:
```
systemctl restart zigbee2mqtt.service
systemctl status zigbee2mqtt.service
```

3- Edit Home Assistant configration.yaml

Adding:
```
nano /home/homeassistant/.homeassistant/configuration.yaml
```
mqtt:
  discovery: true
  broker: 10.0.0.234  # Remove if you want to use builtin-in MQTT broker
  birth_message:
    topic: 'hass/status'
    payload: 'online'
  will_message:
    topic: 'hass/status'
    payload: 'offline'
```


