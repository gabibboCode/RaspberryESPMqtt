# RaspberryESPMqtt

All steps to install a working Moquitto broker on RaspberryPi

# Install Raspbian

- Download Rasbian Lite from https://www.raspberrypi.org/downloads/raspbian/
- Rename downloaded *.img -> image.img
-   diskutil list
- find sd card disk#
-   diskutil unmountDisk /dev/disk#
-   sudo dd bs=1m if=image.img of=/dev/rdisk
-   cp wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant.conf
