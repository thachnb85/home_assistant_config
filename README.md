Travis CI ![alt text](https://travis-ci.org/thachnb85/home_assistant_config.svg?branch=master "Travis CI Result")

![](https://github.com/thachnb85/home_assistant_config/blob/master/HA.png)

# 1. Hardware Overview

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
- Wyze V2 with rtsp firmware: https://support.wyzecam.com/hc/en-us/articles/360026245231-Wyze-Cam-RTSP

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
/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"
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
# 2. Setup zigbee2mqtt.io
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

```
nano /home/homeassistant/.homeassistant/configuration.yaml
```
Adding:
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


# 3. Adding new device
Stop the service
```
systemctl stop zigbee2mqtt.service
```

Navigate to zigbee2mqtt folder then start it manually to see the log:
```
cd  /opt/zigbee2mqtt
npm start
```

Now pairing new device, for example Aqara door sensor: press reset button for 5 secs, then keeps pressing reset button every second to keep it alive, now we can see from log

```
 zigbee2mqtt:info 10/18/2019, 11:05:19 PM MQTT publish: topic 'homeassistant/binary_sensor/0x00158d00029b063b/contact/config', payload '{"payload_on":false,"payload_off":true,"value_template":"{{ value_json.contact }}","device_class":"door","state_topic":"zigbee2mqtt/0x00158d00029b063b","json_attributes_topic":"zigbee2mqtt/0x00158d00029b063b","name":"0x00158d00029b063b_contact","unique_id":"0x00158d00029b063b_contact_zigbee2mqtt","device":{"identifiers":["zigbee2mqtt_0x00158d00029b063b"],"name":"0x00158d00029b063b","sw_version":"Zigbee2mqtt 1.6.0","model":"MiJia door & window contact sensor (MCCGQ01LM)","manufacturer":"Xiaomi"},"availability_topic":"zigbee2mqtt/bridge/state"}'
  zigbee2mqtt:info 10/18/2019, 11:05:19 PM MQTT publish: topic 'homeassistant/sensor/0x00158d00029b063b/battery/config', payload '{"unit_of_measurement":"%","device_class":"battery","value_template":"{{ value_json.battery }}","state_topic":"zigbee2mqtt/0x00158d00029b063b","json_attributes_topic":"zigbee2mqtt/0x00158d00029b063b","name":"0x00158d00029b063b_battery","unique_id":"0x00158d00029b063b_battery_zigbee2mqtt","device":{"identifiers":["zigbee2mqtt_0x00158d00029b063b"],"name":"0x00158d00029b063b","sw_version":"Zigbee2mqtt 1.6.0","model":"MiJia door & window contact sensor (MCCGQ01LM)","manufacturer":"Xiaomi"},"availability_topic":"zigbee2mqtt/bridge/state"}'
  zigbee2mqtt:info 10/18/2019, 11:05:19 PM MQTT publish: topic 'homeassistant/sensor/0x00158d00029b063b/linkquality/config', payload '{"unit_of_measurement":"-","value_template":"{{ value_json.linkquality }}","state_topic":"zigbee2mqtt/0x00158d00029b063b","json_attributes_topic":"zigbee2mqtt/0x00158d00029b063b","name":"0x00158d00029b063b_linkquality","unique_id":"0x00158d00029b063b_linkquality_zigbee2mqtt","device":{"identifiers":["zigbee2mqtt_0x00158d00029b063b"],"name":"0x00158d00029b063b","sw_version":"Zigbee2mqtt 1.6.0","model":"MiJia door & window contact sensor (MCCGQ01LM)","manufacturer":"Xiaomi"},"availability_topic":"zigbee2mqtt/bridge/state"}'
```
Now we can see the device information in Home Assistant web interface > Configuration > Devices
  
```
nano /opt/zigbee2mqtt/data/configuration.yaml
```
Found
```
devices:
  '0x00158d00029b063b':
    friendly_name: '0x00158d00029b063b'
    retain: false
```
We can rename `0x00158d00029b063b`  to `front_stormdoor`, this is the name for easy configuration, we can get the entity name easily from Home Assistent web interface > Configuration > Devices

Now we can go back to Home Assistant and Add this device into its configuration.yaml.
Wherever we want, for example in Sensors section.

Then use the entity name: `binary_sensor.0x00158d00029b063b_contact` to confg HA UI.

# 4. Force Update Sensors

created folder /config/python_scripts
added python_script: to configuration.yaml
Created script: force_update_state.py

````
#pass entity_id as argument from call
sensor = data.get('entity_id')

#read old state
oldstate = hass.states.get(sensor)

#write old state to entity and force update to record database
hass.states.set(sensor, oldstate.state , oldstate.attributes, force_update=True)
````

Then create automation for entity we need to update, for example alarm sensor
```
- alias: 'Uppdate alarm sensor'
  trigger:
    platform: time_pattern
    seconds: '/4'
  action:  
    - data:
        entity_id: sensor.alarm_active
      service: python_script.force_update_state
```

# 5. Advanced Configuration
- Tracking state changes of a sensor: https://community.home-assistant.io/t/timestamp-of-when-a-sensor-switches-to-specific-state/73551/2

By adding new sensor sql which querries directly from database
```
- platform: sql
  queries:
    - name: front door last closed
      query: "select last_changed from states where entity_id = 'binary_sensor.0x00158d000273bbe4_contact' and state = 'on' order by last_changed desc;"
      column: 'last_changed'
      value_template: "{{ as_timestamp(value + 'Z') | timestamp_custom('%a %b %-d %-I:%M%p')}}"
```
Notice that `limit 1 ` is automatically added by HA.
Adding 'Z' for converting to local time zone.

# 6. How to remove an entity
Sometimes you want to remove a sensor, and repair it, for example connect to the router instead of the coordinator


1/ Find the device ID from Configuration> Integrations > MQTT: configuration.yaml
then remove devices from Configuration Entity Registry

2/ ssh into home assistant > stop home assistant

3/ Remove device from bridge
```
mosquitto_pub -h 10.0.0.234 -t "zigbee2mqtt/bridge/config/remove" -m "0x00158d00029bf0c3"
```

List all topics:
```
mosquitto_sub -h 10.0.0.234 -v -d -R -t '#' | grep 0x00158d00029bf0c3
```

Then remove the configs:
```
mosquitto_pub -h 10.0.0.234 -r -n -t "homeassistant/binary_sensor/0x00158d00029bf0c3/contact/config"
mosquitto_pub -h 10.0.0.234 -r -n -t "homeassistant/sensor/0x00158d00029bf0c3/battery/config"
mosquitto_pub -h 10.0.0.234 -r -n -t "homeassistant/sensor/0x00158d00029bf0c3/linkquality/config"
```
4/ Remove device from .storage/core.device_registry
```
cat .storage/core.device_registry | grep 0x00158d00029bf0c3
```
Then find and remove entity 0x00158d00029bf0c3 if has
```
nano .storage/core.device_registry
```

# 7. Trouble Shooting
There are most common issues when I update HA:
### Update ffmpeg
Simple solution is to install from source, for example version 3.2 is required.
```
sudo apt -y install \
    autoconf \
    automake \
    build-essential \
    cmake \
    libass-dev \
    libfreetype6-dev \
    libjpeg-dev \
    libtheora-dev \
    libtool \
    libvorbis-dev \
    libx264-dev \
    pkg-config \
    wget \
    yasm \
    zlib1g-dev

wget http://ffmpeg.org/releases/ffmpeg-3.2.tar.bz2
tar -xjf ffmpeg-3.2.tar.bz2
cd ffmpeg-3.2

./configure --disable-static --enable-shared --disable-doc
make
sudo make install
```

Test if we can see the lib copied to /usr/local/lib by:
```
sudo find / -name libavdevice.so.*
```

We need to update library path:
```
nano /etc/ld.so.conf
```

Adding this line:
```
/usr/local/lib
```
Then run
```
sudo ldconfig
```

### Timezone
HA should be configured with correct timezone for automations working properly.
- In configuration.yaml, check time_zone file, should be full name: America/Toronto
- If we create sensor template from SQL, the time is UTC but it doesn't have `+00:00` at the end, need to append and convert to local time zone, for example:
```
- platform: sql
  scan_interval: 2
  queries:
    - name: front door last opened
      query: "select last_changed from states where entity_id = 'binary_sensor.0x00158d000273bbe4_contact' and state = 'on' order by last_changed desc;"
      column: 'last_changed'
      value_template: "{{ as_timestamp(value~'+00:00') | timestamp_custom('%a %b %-d %-I:%M%p')}}"
```

# 8. Migration to new hardware guide
1. Install HA, MQTT, Zigbee2MQTT

2. Fixes config files to use new IP of MQTT

3. Copy the whole HA config: /home/homeassistant/.homeassistant to new server, dont forget to chmod and chown to homeassistant user.

4. Copy the whole Zigbee2Mqtt config: /opt/zigbee2mqtt/data to new server, dont forget to chmod and chown to homeassistant user.
- Delete database.db in data folder to get fresh start.

5. Repairing Router first, press SW2 button 5 seconds, then move it closer to the Coordinator, light off then change to slow flashing every 3 secs, check logs to make sure it connected.
http://ptvo.info/cc2531-based-router-firmware-136/

6. For repairing sensors, press reset on each sensors.


# 9. Backup script
We need to backup home-assistant and zigbee2mqtt data.

```
cd ~
rm -rdf backup
mkdir backup
mv ha.tar.gz backup
mv data.tar.gz backup

systemctl stop home-assistant.service
systemctl stop zigbee2mqtt.service

cd /home/homeassistant/
tar -zcvf ha.tar.gz .homeassistant
mv ha.tar.gz ~

cd /opt/zigbee2mqtt
tar -zcvf data.tar.gz data
mv data.tar.gz ~

systemctl start home-assistant.service
systemctl start zigbee2mqtt.service
```
