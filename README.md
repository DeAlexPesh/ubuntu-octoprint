## Ubuntu OCTOPRINT (Python3)

<pre>sudo apt -y update && sudo apt -y dist-upgrade

sudo apt -y install net-tools wireless-tools network-manager rfkill git libyaml-dev build-essential
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

`REBOOT:       sudo shutdown -r now`</br>
`SHUTDOWN:     sudo shutdown -h now`

<pre>sudo visudo -f /etc/sudoers.d/octoprint-shutdown
<i>
octoprint ALL=NOPASSWD:/sbin/shutdown
octo ALL=NOPASSWD:/sbin/shutdown
</i></pre>

`OCTORESTART:  sudo service octoprint restart`

<pre>sudo visudo -f /etc/sudoers.d/octoprint-service
<i>
octoprint ALL=NOPASSWD:/usr/sbin/service octoprint restart
octo ALL=NOPASSWD:/usr/sbin/service octoprint restart
</i></pre>

`https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default`
<pre>wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default && \
sudo mv octoprint.default /etc/default/octoprint && \
sudo nano /etc/default/octoprint
<i>
OCTOPRINT_USER=octoprint
BASEDIR=/opt/octoprint/.octoprint
CONFIGFILE=/opt/octoprint/.octoprint/config.yaml
DAEMON=/opt/octoprint/venv3/bin/octoprint
</i></pre>

`https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init`
<pre>wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init && \
sudo mv octoprint.init /etc/init.d/octoprint && \
sudo chmod +x /etc/init.d/octoprint && \
sudo update-rc.d octoprint defaults && \
sudo service octoprint start
</pre>

- #### OCTOPRINT CuraEngine Legacy
`https://github.com/OctoPrint/OctoPrint-CuraLegacy/archive/master.zip`

<pre>sudo apt install cura-engine

Path: <i>/usr/bin/CuraEngine</i>
</pre>

- #### OCTOPRINT Macro
`https://github.com/mike1pol/octoprint_macro/archive/master.zip`
<details>
<summary>HOME X Y Z</summary>
<pre><i>G28 ;Парковка
M84 ;Отключить моторы
</i></pre>
</details>

<details>
<summary>HOME X Y</summary>
<pre><i>G28 X Y ;Парковка
M84 ;Отключить моторы
</i></pre>
</details>

<details>
<summary>PID AUTOTUNE</summary>
<pre><i>G28 ;Парковка
G91 ;Относительные координаты
M83 ;Относительные координаты экструдера
M104 S220 ;Ожидание нагрева до 220 градусов
M109 S220 ;Удержание нагрева до 220 градусов
M106 S160 ;Включить обдув
G1 Z10 E-5 F4500 ;Поднять ось Z на 10 мм и ретракт
M400 ;Ожидание завершения
M303 E0 S230 C8 ;Калибровка экструдера
M106 S0 ;Выключить обдув
M303 E-1 S110 C8 ;Калибровка стола
M500 ;Сохранить в EEPROM
M82 ;Абсолютные координаты экструдера
G90 ;Абсолютные координаты
G28 ;Парковка
M84 ;Отключить моторы
M117 Done! ;Вывести текст
M300 S200 P1000 ;Проиграть звук
</i></pre>
</details>

<details>
<summary>RETRACT</summary>
<pre><i>G28 ;Парковка
G91 ;Относительные координаты
M83 ;Относительные координаты экструдера
G0 Z5 F4500 ;Поднять ось Z на 5 мм
M104 S220 ;Ожидание нагрева до 220 градусов
M109 S220 ;Удержание нагрева до 220 градусов
G0 E-800 F300 ;Ретракт 800 мм со скоростью 300мм/м
M400 ;Ожидание завершения
M104 S0 ;Сброс температуры
G92 E0 ;Сброс длины
M82 ;Абсолютные координаты экструдера
G90 ;Абсолютные координаты
M117 Done! ;Вывести текст
M300 S200 P1000 ;Проиграть звук
</i></pre>
</details>

<details>
<summary>EXTRUDE</summary>
<pre><i>G28 ;Парковка
G91 ;Относительные координаты
M83 ;Относительные координаты экструдера
G0 Z30 F4500 ;Поднять ось Z на 30 мм
M104 S220 ;Ожидание нагрева до 220 градусов
M109 S220 ;Удержание нагрева до 220 градусов
M300 S200 P100 ;Проиграть звук
M300 S200 P1000
G4 S10 ;Ожидание 10 секунд
G0 E800 F300 ;Выдавить 800 мм со скоростью 300мм/м
M400 ;Ожидание завершения
M104 S0 ;Сброс температуры
G92 E0 ;Сброс длины
M82 ;Абсолютные координаты экструдера
G90 ;Абсолютные координаты
M117 Done! ;Вывести текст
M300 S200 P1000 ;Проиграть звук
</i></pre>
</details>

- #### OCTOPRINT plugins shell install (example)

<pre>sudo su - octoprint
source venv/bin/activate
pip install "https://github.com/.../master.zip"
exit
</pre>

- #### OCTOPRINT Web-camera

<pre>sudo apt -y install snapd
sudo nano /etc/environment
<i>
:/snap/bin
</i>
sudo snap install mjpg-streamer && sudo snap connect mjpg-streamer:camera
sudo apt -y install v4l-utils && v4l2-ctl --list-devices
</pre>

<pre>sudo nano /usr/local/bin/stream.sh && sudo chmod +x /usr/local/bin/stream.sh
<i>
#!/bin/bash
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
</i></pre>

<pre>sudo nano /etc/systemd/system/stream.service
<i>
[Unit]
Description=Stream
After=network.target

[Service]
ExecStart=/usr/local/bin/stream.sh start
ExecStop=/usr/local/bin/stream.sh stop
Restart=on-failure

[Install]
WantedBy=default.target
</i></pre>

<pre>sudo visudo -f /etc/sudoers.d/camera-service
<i>
octoprint ALL=NOPASSWD:/usr/sbin/service stream start
octoprint ALL=NOPASSWD:/usr/sbin/service stream restart
octoprint ALL=NOPASSWD:/usr/sbin/service stream stop
octo ALL=NOPASSWD:/usr/sbin/service stream start
octo ALL=NOPASSWD:/usr/sbin/service stream restart
octo ALL=NOPASSWD:/usr/sbin/service stream stop
</i></pre>

<pre>sudo nano /opt/octoprint/.octoprint/config.yaml
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

<pre>sudo systemctl daemon-reload && sudo reboot now
</pre>

`http://octoprint:8080/?action=stream`</br>
`http://octoprint:8080/?action=snapshot`

- #### OCTOPRINT GCodes

<details>
<summary>BEFORE PRINT JOB STARTS</summary>
<pre><i>M300 S200 P100
</i></pre>
</details>

<details>
<summary>AFTER PRINT JOB COMPLETES</summary>
<pre><i>M300 S200 P100
</i></pre>
</details>

<details>
<summary>AFTER PRINT JOB IS CANCELLED</summary>
<pre><i>M84 ;Отключить двигатели
{% snippet 'disable_hotends' %}
{% snippet 'disable_bed' %}
M106 S0 ;Выключить обдув
G90 ;Абсолютные координаты
G28 X Y ;Парковка
</i></pre>
</details>

<details>
<summary>AFTER PRINT JOB IS PAUSED</summary>
<pre><i>{% if pause_position.x is not none %}
G91 ;Относительные координаты
M83 ;Относительные координаты экструдера
G1 Z+5 E-5 F4500 ;Поднятие сопла и ретракт
M82 ;Абсолютные координаты экструдера
G90 ;Абсолютные координаты
G28 X Y ;Парковка
{% endif %}
</i></pre>
</details>

<details>
<summary>BEFORE PRINT JOB IS RESUMED</summary>
<pre><i>{% if pause_position.x is not none %}
M83 ;Относительные координаты экструдера
G1 E-5 F4500 ;Ретракт
G1 E5 F4500 ;Возврат ретракта
M82 ;Абсолютные координаты экструдера
G90 ;Абсолютные координаты
G92 E{{ pause_position.e }} ;Возврат метража
G1 X{{ pause_position.x }} Y{{ pause_position.y }} Z{{ pause_position.z }} F4500 ;Возврат осей
{% if pause_position.f is not none %}G1 F{{ pause_position.f }}{% endif %} ;Возврат экструдера
{% endif %}
</i></pre>
</details>

<details>
<summary>AFTER CONNECTION TO PRINTER IS ESTABLISHED</summary>
<pre><i>M501 ;Чтение EEPROM
M300 S200 P100
</i></pre>
</details>

<details>
<summary>BEFORE CONNECTION TO PRINTER IS CLOSED</summary>
<pre><i>M300 S200 P100
</i></pre>
</details>

- #### Kiosk

<pre>sudo apt -y install --no-install-recommends chromium-browser && \
sudo apt -y install getty-run
</pre>

<pre>sudo nano /lib/systemd/system/getty@.service
<i>
[Service]
...
ExecStart=-/sbin/agetty -a !!!USERNAME!!! --noclear %I $TERM
...
</i></pre>

<pre>sudo apt -y install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox
</pre>

<pre>sudo nano /etc/xdg/openbox/autostart
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

<pre>cd && sudo nano .bash_profile
<i>
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx
</i></pre>

- #### Matrix screensaver

<pre>sudo apt -y install cmatrix
</pre>

<pre>cd && sudo nano .bash_profile
<i>
if [ -n "$BASH_VERSION" ]; then
  if [ -f "$HOME/.bashrc" ]; then
    . "$HOME/.bashrc"
  fi
fi
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && ms
</i></pre>

- #### Alias for Kiosk and Matrix

<pre>sudo nano ~/.bash_aliases && source ~/.bashrc
<i>
alias ms='cmatrix -s -b'
alias xs='startx'
</i></pre>

<pre>sudo nano ~/.bashrc
<i>
if [ -f ~/.bash_aliases ]; then
  . ~/.bash_aliases
fi
</i></pre>
