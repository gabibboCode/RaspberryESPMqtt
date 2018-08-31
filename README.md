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
sudo usermod -aG sudo mosquitto
```
Exit and login as mosquitto user
```
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
sudo journalctl -u mosquitto
```
