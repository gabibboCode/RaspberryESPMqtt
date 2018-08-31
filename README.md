# RaspberryESPMqtt

All steps to install a working Moquitto broker on RaspberryPi

## Install Raspbian, enable wifi and ssh

- Download Rasbian Lite from https://www.raspberrypi.org/downloads/raspbian/
- Rename downloaded *.img -> image.img
```
diskutil list
```
- find sd card disk#
```
diskutil unmountDisk /dev/disk#
sudo dd bs=1m if=image.img of=/dev/rdisk
cp ./wpa_supplicant.conf /Volumes/boot/wpa_supplicant.conf
touch /Volumes/boot/ssh
ssh pi@raspberrypi.local 
raspberry
```

## Install NodeJS
```
cd ~
wget https://nodejs.org/dist/v8.11.4/node-v8.11.4-linux-armv6l.tar.xz
tar -xf node-v8.11.4-linux-armv6l.tar.xz
rm -rf node-v8.11.4-linux-armv6l.tar.xz
cd node-v8.11.4-linux-armv6l
sudo cp -R * /usr/local/
cd /usr/bin
sudo ln -s /usr/local/bin/node node
cd ~
rm -rf ./node-v8.11.4-linux-armv6l
node -v
```

## Add Mosquitto user
```
sudo adduser mosquitto
sudo adduser mosquitto sudo
exit
ssh mosquitto@raspberrypi.local 
raspberry
```

## Install Mosquitto
```
sudo apt-get install mosquitto mosquitto-clients
```
if you have errors
  ```
  wget http://repo.mosquitto.org/debian/mosquitto-repo.gpg.key
  sudo apt-key add mosquitto-repo.gpg.key
  rm mosquitto-repo.gpg.key
  cd /etc/apt/sources.list.d/
  sudo wget http://repo.mosquitto.org/debian/mosquitto-stretch.list
  sudo apt-get update
  sudo apt-get install mosquitto mosquitto-clients
  ```
then add configurations
```
sudo nano /etc/mosquitto/conf.d/mosquitto.conf

# Config file for mosquitto
#
# See mosquitto.conf(5) for more information.
user mosquitto
max_queued_messages 200
message_size_limit 0
allow_zero_length_clientid true
allow_duplicate_messages false
listener 1883
autosave_interval 900
autosave_on_changes false
persistence true
persistence_file mosquitto.db
allow_anonymous true
password_file /etc/mosquitto/passwd
require_certificate false
# End Config file

sudo mosquitto_passwd -c /etc/mosquitto/passwd pi
sudo systemctl restart mosquitto
sudo systemctl status mosquitto
sudo journalctl -fu mosquitto
```


# Install Homebridge

## Add homebridge user
```
sudo adduser homebridge
sudo adduser homebridge sudo
exit
ssh homebridge@raspberrypi.local 
raspberry
```

## Homebridge
```
sudo apt-get install libavahi-compat-libdnssd-dev
sudo npm install -g --unsafe-perm homebridge
```
## Plugin
```
sudo npm install -g homebridge-mqtt-switch-tasmota

sudo nano /home/homebridge/.homebridge/config.json
{
  "bridge": {
    "name": "Homebridge",
    "username": "BA:7C:B0:C6:69:9A",
    "port": 51826,
    "pin": "918-19-468"
  },
  "description": "SmartHome with Homebridge",
  "platforms": [],
  "accessories": [
    {
      "accessory": "mqtt-switch-tasmota",
      "switchType": "outlet",
      "name": "Stereo mansarda",
      "url": "mqtt://192.168.1.9",
      "username": "homebridge",
      "password": "89b0d1",
      "topics": {
        "statusGet": "Home/sonoff_stereo/stat/POWER",
        "statusSet": "Home/sonoff_stereo/cmnd/POWER",
        "stateGet": "Home/sonoff_stereo/tele/STATE"
      },
      "onValue": "ON",
      "offValue": "OFF",
      "activityTopic": "Home/sonoff_stereo/tele/LWT",
      "activityParameter": "Online",
      "startCmd": "Home/sonoff_stereo/cmnd/TelePeriod",
      "startParameter": "60",
      "manufacturer": "ITEAD",
      "model": "Sonoff",
      "serialNumberMAC": ""
    }
  ]
}
```

## Autostart with systemd

```
which homebridge
sudo nano /etc/default/homebridge

# Defaults / Configuration options for homebridge
# The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)
HOMEBRIDGE_OPTS=-U /var/homebridge

# If you uncomment the following line, homebridge will log more 
# You can display this via systemd's journalctl: journalctl -f -u homebridge
# DEBUG=*

sudo nano /etc/systemd/system/homebridge.service
[Unit]
Description=Node.js HomeKit Server 
After=syslog.target network-online.target

[Service]
Type=simple
User=homebridge
EnvironmentFile=/etc/default/homebridge
ExecStart=/usr/bin/homebridge $HOMEBRIDGE_OPTS
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target

sudo mkdir /var/homebridge
sudo cp ~/.homebridge/config.json /var/homebridge/
sudo cp -r ~/.homebridge/persist /var/homebridge
sudo chmod -R 0777 /var/homebridge
sudo systemctl daemon-reload
sudo systemctl enable homebridge
```
