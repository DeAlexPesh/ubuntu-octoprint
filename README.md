## Prepare Ubuntu 20 Server
```bash
sudo apt update && sudo apt -y dist-upgrade

sudo dpkg-reconfigure cloud-init // deselect all

sudo apt purge cloud-init && \
sudo rm -rf {/var/lib/cloud,/etc/cloud} && \
sudo systemctl disable iscsid.service open-iscsi.service && \
sudo apt remove open-iscsi && \
sudo systemctl show -p WantedBy network-online.target
```
```bash
sudo mv /etc/netplan/00-installer-config.yaml /etc/netplan/01-netcfg.yaml && \
sudo nano /etc/netplan/01-netcfg.yaml && \
sudo netplan apply

network:
  version: 2
# renderer: networkd
  ethernets:
    enp0s3:
#     optional: true
      dhcp4: true
#     dhcp6: false
      dhcp-identifier: mac
```
```bash
# if need disabled IPv6
sudo nano /etc/default/grub && sudo update-grub
GRUB_CMDLINE_LINUX_DEFAULT="... ipv6.disable=1"
```
```bash
sudo apt install auditd glances && \
sudo sed -i "s|-b .*|## \0\n-b 15000|" /etc/audit/rules.d/audit.rules && \
sudo service auditd restart && sudo auditctl -s && sudo service auditd status
```
```bash
sudo nano /etc/sysctl.conf && sudo sysctl -p

# swappiness
vm.swappiness=10
vm.vfs_cache_pressure=1000
```
```bash
sudo systemctl reset-failed && \
sudo journalctl --rotate && \
sudo journalctl --vacuum-time=1s && \
sudo systemctl restart systemd-journald
```

## Ubuntu OCTOPRINT (Python3)
```bash
sudo apt -y update && sudo apt -y dist-upgrade

sudo apt -y install net-tools wireless-tools network-manager rfkill git libyaml-dev build-essential
// sudo apt -y install virtualbox-guest-additions-iso
```

- #### Wi-Fi drivers
```bash
lspci -knn | grep Net -A2
sudo update-pciids

# BCM4312
sudo apt install firmware-b43-installer -y && sudo reboot now

sudo rfkill list all 
sudo rfkill unblock all

iwconfig
```

- #### Wi-Fi connection
```bash
nmcli d
nmcli r wifi on
nmcli d wifi list
```
<pre>
<b>sudo nano /usr/local/bin/wlan-connect.sh && sudo chmod +x /usr/local/bin/wlan-connect.sh</b>
<i>
#!/bin/bash
nmcli d wifi list && \
read -er -p 'SSID: ' WSSID && \
read -er -p 'PASS: ' WPASS && \
sudo nmcli d wifi connect "$WSSID" password "$WPASS" && \
nmcli d
</i></pre>

```bash
wlan-connect.sh


```

- #### Case settings (notebooks)
```bash
# man logind.conf 
sudo apt -y install pm-utils && \
sudo cp /etc/systemd/logind.conf /etc/systemd/logind.conf.back
```
<pre>
<b>sudo nano /etc/systemd/logind.conf</b>

<i># do nothing then noteboock is closed</i>
<i>HandleLidSwitch=ignore</i>
</pre>

```bash
# set period backlight is off (sec)
sudo nano /etc/default/grub && sudo update-grub
GRUB_CMDLINE_LINUX_DEFAULT="... quiet consoleblank=10"

sudo systemctl restart systemd-logind.service
```
- #### OCTOPRINT install ( http://ip:5000 )

```bash
# Ubuntu 16

sudo apt-key adv --refresh-keys --keyserver keyserver.ubuntu.com
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update && \
sudo apt -y install python3.9 python3.9-distutils
// sudo apt remove python3.5-minimal -y
ln -sf /usr/bin/python3.9 /usr/local/bin/python3

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
python3.9 get-pip.py && \
rm get-pip.py

which python3 && python3 --version && pip -V
```

```bash
sudo apt -y install python3-pip python3-dev python3-setuptools python3-virtualenv python3-venv

sudo useradd -m -d /opt/octoprint -s /bin/bash octoprint && \
sudo usermod -a -G tty octoprint && \
sudo usermod -a -G dialout octoprint && \
sudo su - octoprint
```
```python
virtualenv --python=/usr/bin/python3 venv3
source venv3/bin/activate
pip install pip --upgrade
pip install OctoPrint

# curl -LO https://bootstrap.pypa.io/get-pip.py && python get-pip.py --user
# python -m pip install --user --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org --upgrade pip
# python -m pip install --user --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org OctoPrint

deactivate && exit
```

- #### OCTOPRINT actions
`REBOOT: sudo shutdown -r now`</br>
`SHUTDOWN: sudo shutdown -h now`
<pre>
<b>sudo visudo -f /etc/sudoers.d/octoprint-shutdown</b>
<i>
octoprint ALL=NOPASSWD:/sbin/shutdown
octo ALL=NOPASSWD:/sbin/shutdown
</i></pre>

`OCTORESTART: sudo service octoprint restart`
<pre>
<b>sudo visudo -f /etc/sudoers.d/octoprint-service</b>
<i>
octoprint ALL=NOPASSWD:/usr/sbin/service octoprint restart
octo ALL=NOPASSWD:/usr/sbin/service octoprint restart
</i></pre>
  
[`octoprint.default`](https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default)

```bash
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default && \
sudo mv octoprint.default /etc/default/octoprint
```
<pre>
<b>sudo nano /etc/default/octoprint</b>
<i>
OCTOPRINT_USER=octoprint
BASEDIR=/opt/octoprint/.octoprint
CONFIGFILE=/opt/octoprint/.octoprint/config.yaml
DAEMON=/opt/octoprint/venv3/bin/octoprint
</i></pre>

[`octoprint.init`](https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init)

```bash
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init && \
sudo mv octoprint.init /etc/init.d/octoprint && \
sudo chmod +x /etc/init.d/octoprint && \
sudo update-rc.d octoprint defaults && \
sudo service octoprint start
```

- #### OCTOPRINT CuraEngine Legacy - [GIT](https://github.com/OctoPrint/OctoPrint-CuraLegacy/archive/master.zip)
```bash
sudo apt install cura-engine


```
<pre><b>Path:</b> <i>/usr/bin/CuraEngine</i></pre>

- #### OCTOPRINT Macro - [GIT](https://github.com/mike1pol/octoprint_macro/archive/master.zip)
<details>
<summary>HOME X Y Z</summary>
  
```gcode
G28 ;????????????????
M84 ;?????????????????? ????????????
```
</details>

<details>
<summary>HOME X Y</summary>
  
```gcode
G28 X Y ;????????????????
M84 ;?????????????????? ????????????
```
</details>

<details>
<summary>PID AUTOTUNE</summary>
  
```gcode
G28 ;????????????????
G91 ;?????????????????????????? ????????????????????
M83 ;?????????????????????????? ???????????????????? ????????????????????
M104 S220 ;???????????????? ?????????????? ???? 220 ????????????????
M109 S220 ;?????????????????? ?????????????? ???? 220 ????????????????
M106 S160 ;???????????????? ??????????
G1 Z10 E-5 F4500 ;?????????????? ?????? Z ???? 10 ???? ?? ??????????????
M400 ;???????????????? ????????????????????
M303 E0 S230 C8 ;???????????????????? ????????????????????
M106 S0 ;?????????????????? ??????????
M303 E-1 S110 C8 ;???????????????????? ??????????
M500 ;?????????????????? ?? EEPROM
M82 ;???????????????????? ???????????????????? ????????????????????
G90 ;???????????????????? ????????????????????
G28 ;????????????????
M84 ;?????????????????? ????????????
M117 Done! ;?????????????? ??????????
M300 S200 P1000 ;?????????????????? ????????
```
</details>

<details>
<summary>RETRACT</summary>
  
```gcode
G28 ;????????????????
G91 ;?????????????????????????? ????????????????????
M83 ;?????????????????????????? ???????????????????? ????????????????????
G0 Z5 F4500 ;?????????????? ?????? Z ???? 5 ????
M104 S220 ;???????????????? ?????????????? ???? 220 ????????????????
M109 S220 ;?????????????????? ?????????????? ???? 220 ????????????????
G0 E-800 F300 ;?????????????? 800 ???? ???? ?????????????????? 300????/??
M400 ;???????????????? ????????????????????
M104 S0 ;?????????? ??????????????????????
G92 E0 ;?????????? ??????????
M82 ;???????????????????? ???????????????????? ????????????????????
G90 ;???????????????????? ????????????????????
M117 Done! ;?????????????? ??????????
M300 S200 P1000 ;?????????????????? ????????
```
</details>

<details>
<summary>EXTRUDE</summary>
  
```gcode
G28 ;????????????????
G91 ;?????????????????????????? ????????????????????
M83 ;?????????????????????????? ???????????????????? ????????????????????
G0 Z30 F4500 ;?????????????? ?????? Z ???? 30 ????
M104 S220 ;???????????????? ?????????????? ???? 220 ????????????????
M109 S220 ;?????????????????? ?????????????? ???? 220 ????????????????
M300 S200 P100 ;?????????????????? ????????
M300 S200 P1000
G4 S10 ;???????????????? 10 ????????????
G0 E800 F300 ;???????????????? 800 ???? ???? ?????????????????? 300????/??
M400 ;???????????????? ????????????????????
M104 S0 ;?????????? ??????????????????????
G92 E0 ;?????????? ??????????
M82 ;???????????????????? ???????????????????? ????????????????????
G90 ;???????????????????? ????????????????????
M117 Done! ;?????????????? ??????????
M300 S200 P1000 ;?????????????????? ????????
```
</details>

- #### OCTOPRINT plugins shell install (example)
```bash
sudo su - octoprint


```
```python
source venv/bin/activate
pip install "https://github.com/.../master.zip"
exit
```

- #### OCTOPRINT Web-camera
```bash
sudo apt -y install snapd


```
<pre>
<b>sudo nano /etc/environment</b>

<i>... :/snap/bin</i>
</pre>

```bash
sudo snap install mjpg-streamer && sudo snap connect mjpg-streamer:camera
sudo apt -y install ffmpeg v4l-utils && sudo v4l2-ctl --list-devices
```

<pre>
<b>sudo nano /usr/local/bin/stream.sh && sudo chmod +x /usr/local/bin/stream.sh</b>
<i>
#!/bin/bash
RESOLUTION="640x480"
FRAMERATE="25"
VIDEO="/dev/video0"
MJPG_WEB_ROOT="/tmp"
PORT="8080"
LISTEN="0.0.0.0"
DAEMON="mjpg_streamer"
case "$1" in
  start)
    mjpg-streamer -i "input_uvc.so -d $VIDEO -f $FRAMERATE -r $RESOLUTION -yuv -n -timestamp" -o "output_http.so -w $MJPG_WEB_ROOT -p $PORT"
  ;;
  stop)
    pkill -x ${DAEMON}
  ;;
esac
</i></pre>

<pre>
<b>sudo nano /etc/systemd/system/stream.service</b>
<i>
[Unit]
Description=Stream
After=network.target

[Service]
CPUWeight=50
CPUQuota=50%
IOWeight=50
MemorySwapMax=0
ExecStart=/usr/local/bin/stream.sh start
ExecStop=/usr/local/bin/stream.sh stop
Restart=on-failure

[Install]
WantedBy=default.target
</i></pre>

<pre>
<b>sudo visudo -f /etc/sudoers.d/camera-service</b>

<i>octoprint ALL=NOPASSWD:/usr/sbin/service stream start
octoprint ALL=NOPASSWD:/usr/sbin/service stream restart
octoprint ALL=NOPASSWD:/usr/sbin/service stream stop
octo ALL=NOPASSWD:/usr/sbin/service stream start
octo ALL=NOPASSWD:/usr/sbin/service stream restart
octo ALL=NOPASSWD:/usr/sbin/service stream stop
</i></pre>

<pre>
<b>sudo nano /opt/octoprint/.octoprint/config.yaml</b>
<i>
system:
  actions:
  - name: Camera enable
    action: Camera enable
    command: sudo service stream start
    confirm: false
  - name: Camera disable
    action: Camera disable
    command: sudo service stream stop
    confirm: false
</i></pre>

```bash
sudo systemctl daemon-reload && sudo reboot now


```

`http://octoprint:8080/?action=stream`</br>
`http://octoprint:8080/?action=snapshot`

- #### OCTOPRINT GCodes

<details>
<summary>BEFORE PRINT JOB STARTS</summary>

```gcode
M300 S200 P100
```
</details>

<details>
<summary>AFTER PRINT JOB COMPLETES</summary>
  
```gcode
M300 S200 P100
```
</details>

<details>
<summary>AFTER PRINT JOB IS CANCELLED</summary>
  
```gcode
M84 ;?????????????????? ??????????????????
{% snippet 'disable_hotends' %}
{% snippet 'disable_bed' %}
M106 S0 ;?????????????????? ??????????
G90 ;???????????????????? ????????????????????
G28 X Y ;????????????????
```
</details>

<details>
<summary>AFTER PRINT JOB IS PAUSED</summary>

```gcode
{% if pause_position.x is not none %}
G91 ;?????????????????????????? ????????????????????
M83 ;?????????????????????????? ???????????????????? ????????????????????
G1 Z+5 E-5 F4500 ;???????????????? ?????????? ?? ??????????????
M82 ;???????????????????? ???????????????????? ????????????????????
G90 ;???????????????????? ????????????????????
G28 X Y ;????????????????
{% endif %}
```
</details>

<details>
<summary>BEFORE PRINT JOB IS RESUMED</summary>

```gcode
{% if pause_position.x is not none %}
M83 ;?????????????????????????? ???????????????????? ????????????????????
G1 E-5 F4500 ;??????????????
G1 E5 F4500 ;?????????????? ????????????????
M82 ;???????????????????? ???????????????????? ????????????????????
G90 ;???????????????????? ????????????????????
G92 E{{ pause_position.e }} ;?????????????? ??????????????
G1 X{{ pause_position.x }} Y{{ pause_position.y }} Z{{ pause_position.z }} F4500 ;?????????????? ????????
{% if pause_position.f is not none %}G1 F{{ pause_position.f }}{% endif %} ;?????????????? ????????????????????
{% endif %}
```
</details>

<details>
<summary>AFTER CONNECTION TO PRINTER IS ESTABLISHED</summary>
  
```gcode
M501 ;???????????? EEPROM
M300 S200 P100
```
</details>

<details>
<summary>BEFORE CONNECTION TO PRINTER IS CLOSED</summary>
  
```gcode
M300 S200 P100
```
</details>

- #### Kiosk
```bash
sudo apt -y install --no-install-recommends chromium-browser && \
sudo apt -y install getty-run
```

<pre>
<b>sudo nano /lib/systemd/system/getty@.service</b>
<i>
[Service]
...
ExecStart=-/sbin/agetty -a !!!USERNAME!!! --noclear %I $TERM
...
</i></pre>

<pre>sudo apt -y install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox
</pre>

<pre>
<b>sudo nano /etc/xdg/openbox/autostart</b>
<i>
# Disable any form of screen saver
xset s off
# Disable screen blanking
xset s noblank
# Disable power management
xset -dpms
# Allow quitting the X server with CTRL-ATL-Backspace
setxkbmap -option terminate:ctrl_alt_bksp
setxkbmap -option startx:alt_x
# Start Chromium in kiosk mode
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
chromium-browser --disable-infobars --kiosk 'http://localhost:5000/'
</i></pre>

<pre>
<b>cd && sudo nano .bash_profile</b>
<i>
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx
</i></pre>

- #### Matrix screensaver

<pre>
<b>sudo apt -y install cmatrix && cd && sudo nano .bash_profile</b>
<i>
if [ -n "$BASH_VERSION" ]; then
  if [ -f "$HOME/.bashrc" ]; then
    . "$HOME/.bashrc"
  fi
fi
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && ms
</i></pre>

- #### Alias for Kiosk and Matrix

<pre>
<b>sudo nano ~/.bash_aliases && source ~/.bashrc</b>
<i>
alias ms='cmatrix -s -b'
alias xs='startx'
</i></pre>

<pre>
<b>sudo nano ~/.bashrc</b>
<i>
...
if [ -f ~/.bash_aliases ]; then
  . ~/.bash_aliases
fi
</i></pre>
