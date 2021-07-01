## Ubuntu OCTOPRINT (Python3)

<pre>sudo apt -y update && sudo apt -y dist-upgrade

sudo apt -y install net-tools rfkill wireless-tools network-manager
sudo apt -y install git libyaml-dev build-essential
sudo apt -y install virtualbox-guest-additions-iso
</pre>

- #### Wi-Fi drivers

<pre>lspci -knn | grep Net -A2
sudo update-pciids

# BCM4312
sudo apt install firmware-b43-installer -y && sudo reboot now

sudo rfkill list all 
sudo rfkill unblock all

iwconfig
</pre>

- #### Wi-Fi connection

<pre>nmcli d
nmcli r wifi on
nmcli d wifi list

read -er -p 'SSID: ' WSSID && \
read -er -p 'PASS: ' WPASS && \
sudo nmcli d wifi connect "$WSSID" password "$WPASS" && \
nmcli d
</pre>

- #### Case settings (notebooks)

<pre># man logind.conf 
sudo apt -y install pm-utils && \
sudo cp /etc/systemd/logind.conf /etc/systemd/logind.conf.back && \
sudo nano /etc/systemd/logind.conf

<i>HandleLidSwitch=ignore</i>

sudo systemctl restart systemd-logind.service
</pre>

- #### OCTOPRINT install ( http://ip:5000 )

<pre>sudo apt -y install python3-pip python3-dev python3-setuptools python3-virtualenv python3-venv

sudo useradd -m -d /opt/octoprint -s /bin/bash octoprint && \
sudo usermod -a -G tty octoprint && \
sudo usermod -a -G dialout octoprint && \
sudo su - octoprint

virtualenv --python=/usr/bin/python3 venv3
source venv3/bin/activate
pip install pip --upgrade
pip install OctoPrint
exit
</pre>

- #### OCTOPRINT actions

`REBOOT:       sudo shutdown -r now`

`SHUTDOWN:     sudo shutdown -h now`

`OCTORESTART:  sudo service octoprint restart`

<pre>sudo visudo -f /etc/sudoers.d/octoprint-shutdown
<i>
octoprint ALL=NOPASSWD:/sbin/shutdown
octo ALL=NOPASSWD:/sbin/shutdown
</i></pre>

<pre>sudo visudo -f /etc/sudoers.d/octoprint-service
<i>
octoprint ALL=NOPASSWD:/usr/sbin/service octoprint restart
octo ALL=NOPASSWD:/usr/sbin/service octoprint restart
</i></pre>

<pre># https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init && \
sudo mv octoprint.init /etc/init.d/octoprint
</pre>

<pre># https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default && \
sudo mv octoprint.default /etc/default/octoprint
</pre>

<pre>sudo nano /etc/default/octoprint
<i>
OCTOPRINT_USER=octoprint
BASEDIR=/opt/octoprint/.octoprint
CONFIGFILE=/opt/octoprint/.octoprint/config.yaml
DAEMON=/opt/octoprint/venv3/bin/octoprint
</i></pre>

<pre>sudo chmod +x /etc/init.d/octoprint && \
sudo update-rc.d octoprint defaults && \
sudo service octoprint start
</pre>

- #### OCTOPRINT plugins

<pre>sudo su - octoprint
source venv/bin/activate
pip install "https://github.com/.../master.zip"
exit
</pre>

- ### OCTOPRINT web-camera

<pre><code>sudo apt install snapd -y
sudo nano /etc/environment
:/snap/bin"
</code></pre>

<pre><code>sudo apt install v4l-utils -y
sudo snap install mjpg-streamer
sudo snap connect mjpg-streamer:camera
</code></pre>

<pre><code>v4l2-ctl --list-devices
</code></pre>

<pre><code>sudo nano /usr/local/bin/stream.sh && sudo chmod +x /usr/local/bin/stream.sh
</code></pre>

<pre><code>#!/bin/bash
RESOLUTION="640x480"
FRAMERATE="25"
VIDEO="/dev/video0"
MJPG_WEB_ROOT="/tmp"
PORT="8080"
LISTEN="127.0.0.1"
DAEMON="mjpg_streamer"

case "$1" in
  start)
    mjpg-streamer -i "input_uvc.so -d $VIDEO -f $FRAMERATE -r $RESOLUTION -yuv" -o "output_http.so -w $MJPG_WEB_ROOT -p $PORT"
  ;;
  stop)
    pkill -x ${DAEMON}
  ;;
esac
</code></pre>

<pre><code>sudo nano /etc/systemd/system/stream.service
</code></pre>

<pre><code>[Unit]
Description=Stream
After=network.target

[Service]
ExecStart=/usr/local/bin/stream.sh start
ExecStop=/usr/local/bin/stream.sh stop
Restart=on-failure

[Install]
WantedBy=default.target
</code></pre>

<pre><code>sudo systemctl daemon-reload
</code></pre>

<pre><code>sudo visudo -f /etc/sudoers.d/camera-service
</code></pre>

<pre><code>octoprint ALL=NOPASSWD:/usr/sbin/service stream start
octoprint ALL=NOPASSWD:/usr/sbin/service stream restart
octoprint ALL=NOPASSWD:/usr/sbin/service stream stop
octo ALL=NOPASSWD:/usr/sbin/service stream start
octo ALL=NOPASSWD:/usr/sbin/service stream restart
octo ALL=NOPASSWD:/usr/sbin/service stream stop
</code></pre>

<pre><code>sudo nano /opt/octoprint/.octoprint/config.yaml
</code></pre>

<pre><code>system:
  actions:
  - name: Camera enable
    action: Camera enable
    command: sudo service stream start
    confirm: false
  - name: Camera disable
    action: Camera disable
    command: sudo service stream stop
    confirm: false
</code></pre>

<pre><code>sudo reboot now
</code></pre>

> http://octoprint:8080/?action=stream
> http://octoprint:8080/?action=snapshot

<pre><code>
</code></pre>

<pre><code>
</code></pre>

<pre><code>
</code></pre>

<pre><code>
</code></pre>

<pre><code>
</code></pre>

<pre><code>
</code></pre>
