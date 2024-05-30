# Virtual Presence Robot
## Introducation

This is our summer 2024 work term project in NSCC IT campus, which is a Raspberry Pi-based robot project. Users can remotely control the robot's behavior through a Web interface. These behaviors include real-time camera monitoring and six different movement modes, similar to lunar exploration robots. It contains:

- UV4L-Server
- API-Server
- Motor-Control-Part


![System Architecture Diagram](https://raw.githubusercontent.com/WT-VirtualPresenceRobot/Introduction/main/diagram.png )
<p align="center">System Architecture Diagram</p>

## Usage 
_@Our Groupmembers_

Simply put, create repositories for the parts you're responsible for, and then push them up.
- Local git commit
- Create remote repository in Github here
- Add remote in your IDE, remote name origin and push

**Some command you may need:**
```sh
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global --list
```
**Some tutorial you may need:**<br>
[Using Git with Visual Studio Code](https://www.youtube.com/watch?v=i_23KUAEtUM)<br>
[Editing Code on your Raspberry Pi Remotely with VS Code](https://www.youtube.com/watch?v=jvi1nmKK81Y)<br>

## Installation Guide
### 1.Setting up Wi-Fi connection to eduroam
> To ensure video streaming low latency, we need to connect the RPi and our client to the same LAN network. However, the WiFi coverage of the IoT-D210 is too limited to demonstrate in other parts of the campus. Therefore, it is recommended to connect both your laptop and the RPi to eduroam.

When you are trying to connect to eduroam WiFi on Raspberry Pi,  you will see the WiFi access point is greyed out on Raspberry Pi. It is because Raspberry Pi is using the simple network service on the GUI, which doesn't support WPA2 enterprise WiFi. And eduroam is using WPA2 enterprise.
However, Raspberry Pi is supporting WPA2 enterprise WiFi. And the simplest way to setup the connection on Raspberry Pi is using `wpa_supplicant.conf` file.
Type this in your terminal
```sh
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```
And type these content
```sh
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev update_config=1
country=CA
network={
    ssid="eduroam"
    scan_ssid=1
    proto=RSN
    key_mgmt=WPA-EAP
    eap=РЕАР
    identity="w12345@campus.nscc.ca" password="12345"  #change to your W number and password
    phase1="peaplabel=0"
    phase2="auth=MSCHAPV2"
    priority=8
```
Then Press Ctrl+O to save and Ctrl+X to exit and reboot RPi.

### 2.Installation for UV4L-Server
> This part is responsible for live video streaming and website deployment. The UV4L Server includes a streaming server and a simple HTTP server(for deploying our webpage).


Our RPi is use RPi OS Bullseye, Debian 11, 32-bit.
```sh
curl https://www.linux-projects.org/listing/uv4l_repo/lpkey.asc | sudo apt-key add -
sudo nano /etc/apt/sources.list
```
Paste the following line into the file we just opened
```sh
deb https://www.linux-projects.org/listing/uv4l_repo/raspbian/stretch stretch main
```
Save by pressing Ctrl + O then enter and exit with CtrI + X
Now we are ready to update the system and fetch and install the packages
```sh
sudo apt-get update
sudo apt-get install uv4l uv4l-raspicam
sudo apt-get install uv4l-raspicam-extras uv4l-server uv4l-mjpegstream uv4l-demos uv4l-xmpp-bridge
```
We will be using WebRTC to stream the Camera
```sh
sudo apt-get install uv4l-webrtc
```

At any time you can restart the UV4L service by typing, Especially if you change the configuration, you need to restart the service for it to take effect.
```sh
sudo service uv4l_raspicam restart
```

**Test the camera and UV4l Server**
On your Web-browser on your computer to http://192.168.1.100:8080/ substituting 192.168.1.100 for your Raspberry Pi’s IP address.
If it works. You can do the following configuration.

**Configure the UV4l Server**
We are going to edit the default configuration file of the UV4L server. After making changes, ctrl-o and enter to save, then ctrl-x and enter to exit. Then restart the UV4L server.
```sh
sudo nano /etc/uv4l/uv4l-raspicam.conf
```
Using the arrow keys to navigate the file we will be un-commenting some lines bydeleting the # in front ofthem, and modifying the values
Find the sections below(some are right near the bottom of the file) and make these changes and uncomment by removing the #
```sh
### WebRTC options:
server-option =--enable-webrtc=yes
server-option =--enable-webrtc-datachannels=yes
server-option =--webrtc-datachannel-label=uv4l
server-option =--webrtc-datachannel-socket=/tmp/uv4l.socket
server-option =--enable-webrtc-video=yes
server-option =--enable-webrtc-audio=yes
...
server-option =--webrtc-max-playout-delay=34
...
### These options are specific to the HTTP/HTTPS Server
### serving custom Web pages only:
server-option =--enable-www-server=yes
server-option =--www-root-path=/usr/share/uv4l/demos/robot/  #  You can change this to your own folder
server-option =--www-index-file=index.html
server-option =--www-port=8888  # You can change this
...
server-option =--www-max-queued-connections=8
server-option =--www-max-threads=4
server-option =--www-thread-idle-time=10
...
server-option = --www-webrtc-signaling-path=/webrtc
```
Then you can use http://192.168.1.100:8888/ to visit this web page /usr/share/uv4l/demos/robot/index.html
Next, Create your own project folder (Actually, this step should be done in advance, but it has little impact)
```sh
cd /usr/share/uv4l/demos  # You can change to your location
sudo mkdir robot
cd robot
sudo wget https://github.com/SUMMER2024-WT-VirtualPresenceRobot/API-Server/archive/refs/heads/main.zip #UV4L-Server repo
sudo unzip-j main.zip
```
In your web-browser now to 192.168.1.100:8888 with your IP Raspberry Pi's address. You should see the Robot website. 
Test it by starting the steaming.
Then click stop streaming.
This UV4l server provides a simple HTTP server as well as a streaming server. The reason for using its built-in HTTP server is because we also need to use its WebRTC-signaling functionality.

[Official Installation](https://www.linux-projects.org/uv4l/installation/)

### 3.Installation for API Server
> This part is responsible for remote control the robot. 
The reason it's called API Server is that we need an API Web Server to receive POST requests from the client side, which contain the user's control signal. This signal is then used to control the robot. 
In ***API-Server*** repository
`api.py` is the core program file.
`robot_control.py` is the robotControl class file.
`lcd.py` is just a testing file.

#### First, Create the folder and download the necessay code file.
```sh
cd /usr/share/uv4l/demos/robot  # You can change to your location
sudo mkdir api-server
cd api-server
sudo wget https://github.com/WT-VirtualPresenceRobot/API-Server/archive/refs/heads/main.zip #API-Server repo
sudo unzip-j main.zip
```
#### Install Python3
In terminal
```sh
sudo apt-get update
sudo apt-get install python3
```
#### Install Web Server - Flask
```sh
pip3 install flask flask-sqlalchemy
```
#### Install all the packages
```sh
pip3 install flask_cors, evdev, RPLCD, RPi.GPIO
```
Then you can use this command to run the `api.py`.
```sh
cd path-to-api-server 
sudo python3 api.py
```

### 4. Convert this command into a service
Create service file.

```bash
sudo nano /etc/systemd/system/robotapi.service
```

```bash
[Unit]
Description=Robot API Receive process control signals
After=network.target

[Service]
User=pi # need change according to your own
Group=pi # need change according to your own
WorkingDirectory=/home/pi/VirtualPresenceRobot/api-server # need change according to your own
ExecStart=/bin/bash -c 'sudo python3 /home/pi/VirtualPresenceRobot/robotcam/api.py>
Restart=always
RestartSec=3
Environment="PATH=/home/pi/VirtualPresenceRobot/robotcam/env/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable robotcamapi.service
sudo systemctl start robotcamapi.service
```

### 5. Configure voice streaming
```sh
sudo nano /etc/uv4l/uv4l-raspicam.conf
```
Change to the right index.
```sh
server-option = --webrtc-recdevice-index=4 #start count from 0
```
One way to find this right index is use this command.
```sh
sudo journalctl -u uv4l_raspicam.service -f
```
Check the error message it shows. Test index from 0 to see error message. like in my case, if I choose index is 1. it shows
```python
May 14 15:35:49 pi uv4l[28073]: <notice> [webrtc] Enabling WebRTC Audio Recording Device 'bcm2835 Headphones-USB Stream Output', index 1
May 14 15:35:50 pi uv4l[28073]: ALSA lib pcm_usb_stream.c:508:(_snd_pcm_usb_stream_open) Unknown field hint
May 14 15:35:50 pi uv4l[28073]: ALSA lib pcm_usb_stream.c:508:(_snd_pcm_usb_stream_open) Unknown field hint
```
Compare with the result from this command.
If bcm2835 is index 1.
Then plughw should be 3.
```python
arecord --list-pcms
```
```sh
usbstream: CARD=Headphones
    bcm2835 Headphones
    USB Stream Output
hw: CARD=Device, DEV=0
    USB PnP Sound Device, USB Audio
    Direct hardware device without any conversions 
plughw: CARD=Device, DEV=0  # We need find this one
    USB PnP Sound Device, USB Audio
    Hardware device with all software conversions
```
If you choose the right index, journalctl will show like this.
```sh
May 14 15:39:38 pi uv41[28804]: ‹notice> [webrtcl Enabling WebRTC Audio Recording Device 'USB PnP Sound Device, USB Audio-Direct hardware device without any convers ions', index 3
```

DONE!


## License

MIT

**Free Software, Hell Yeah!**
