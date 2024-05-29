# Virtual Presence Robot
## Introducation

This is our summer 2024 work term project in NSCC IT campus, which is a Raspberry Pi-based robot project. Users can remotely control the robot's behavior through a Web interface. These behaviors include real-time camera monitoring and six different movement modes, similar to lunar exploration robots. It contains:

- UV4L-Server
- API-Server
- Motor-Control-Part

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
### Setting up Wi-Fi connection to eduroam
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
    phase2="auth=MSCHAPV2" -
    priority=8
```
Then Press Ctrl+O to save and Ctrl+X to exit and reboot RPi.

## License

MIT

**Free Software, Hell Yeah!**
