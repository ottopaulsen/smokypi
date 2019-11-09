# smokypi

## Introduction

Smokypi is my IoT server, running on a Raspberry Pi. Since I, for different reasons, have had to reinstall this server multiple times, I am making this repository in order to make it as easy as possible to reinstall the server. The installation instructions below takes you from a brand new Raspberry Pi to a running server with the following services:

* Mosquitto running an MQTT message queue
* Node-RED for orchestrating messages, integrating with Firestore and ThingSpeak
* Weewx for integrating my weather station with MQTT and with Weather Underground
* Logging in via ssh without having to enter password

My Node-RED flow is set up for my different IoT needs, and is of course very specific for myself. Actually I only do this for myself, but I try to make it at least partly useful for others that want to do something similar.


NB! Currently this is work in progres!

## Installation and configuration

### Install a new Raspberry Pi

I use macos for downloading Raspbian and creating a Raspberry Pi memory card.

Download [Raspbian Stretch with desktop](https://www.raspberrypi.org/downloads/raspbian/) and unpack it.
Insert memory card in mac. Find the mount name and the device name:
```
mount
```
Unmount (example name boot):
```
sudo diskutil unmount /Volumes/boot
```
Then install (example device disk2):
```
sudo dd bs=1m if=2019-09-26-raspbian-buster.img of=/dev/rdisk2
```
NB! Remember the “r” in `rdisk2`.
It may take a minute or so, so wait patiently.

The easiest way to enable ssh is to do it now, by creating a file named ssh in the root folder (assuming the memory card is mounted as `boot` on the mac):
```
touch /Volumes/boot/ssh
```
Unmount the memory card (e.g. from Finder) or with
```
diskutil unmount /Volumes/boot
```

Insert the memory card in the RPi and boot with screen, keyboard and mouse attached.
Follow instructions and update the RPi.

Now you can find the ip of the RPi and log in via ssh.

### Change hostname

Change hostname:
```
sudo vi /etc/hosts /etc/hostname
```
Then change the hostname in these two files and reboot. My hostname is `smokypi`


### Enable ssh login without password

On server (RPi):
```
# NB! OnlY copy/paste one command at a time:
mkdir .ssh
cd .ssh
ssh-keygen -t rsa # hit return through prompts
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
rm id_rsa.pub
```
On client:
```
cd .ssh
scp smokypi:.ssh/id_rsa smokypi.rsa
chmod 600 smokypi.rsa
# Do the following only if not already done:
echo "Host smokypi" >> config
echo "Hostname 10.0.0.15" >> config
echo "IdentityFile ~/.ssh/smokypi.rsa" >> config
```
Test:
```
ssh smokypi
```

### Install mosquitto

Mosquitto is an implementation of MQTT, and you will normally need to install it. Running in Docker only is not enough. For example, you need it to generate the password file.

Install Mosquitto and client tools with the following commands:
```
sudo apt-get install -y mosquitto
sudo apt-get install -y mosquitto-clients
```

Secure mosquitto by creating a password file:
```
sudo mosquitto_passwd -c /etc/mosquitto/passwd username
sudo bash -c "echo allow_anonymous false >> /etc/mosquitto/mosquitto.conf"
sudo bash -c "echo password_file /etc/mosquitto/passwd >> /etc/mosquitto/mosquitto.conf"
sudo service mosquitto restart
```
The first command will ask for password to that user.

Changing password can be done like this:
```
sudo mosquitto_passwd -b /etc/mosquitto/passwd user password
sudo service mosquitto restart
```

### Install Node-RED

Install node, npm and Node-RED with the following command:
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

Then install the nodes you need, for example:
```
sudo npm install -g node-red-contrib-thingspeak42
sudo npm install -g node-red-contrib-firebase
sudo npm install -g node-red-dashboard
sudo npm install -g node-red-dashboard
sudo npm install -g node-red-admin
```

NB! The `node-red-admin` install will give error messages, but they seem not to matter.

Enable node-red to start as a service when the Pi is turned on:
```
sudo systemctl enable nodered.service
```

And start it immediately using:
```
sudo service nodered start
```

Enable security by uncommenting the `adminAuth` section in `.node-red/settings.js`, and edit user name and hashed password. Do the same with the `httpNodeAuth` section to secure e.g. Node-RED Dashboard.

Hash password with `node-red-admin hash-pw`.

Restart the service with `sudo service nodered start`.

### Setup git

Copy the public key found in .ssh/authorized_keys to github.

TODO: (Hmm. There is something strange here...  Done above, I think)

```
cd
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
eval "$(ssh-agent -s)"
```

Then pull this repo to the pi-users home directory.



