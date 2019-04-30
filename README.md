# HDMI Pi Display Board

Automated setup scripts and configuration files for creating HDMI video display boards powered by Raspberry Pi 3 Bs (should work for others).

### Prerequisites

You will need to download and burn the latest Raspberry Lite image to an SD card before you begin. This should work with the full desktop version, but you ideally would want to run this without all the extra overhead.

You can find the download page [here](https://www.raspberrypi.org/downloads/raspbian/).

## Getting Started

These instructions will take a clean install of the headless Raspbian and set it up to loop a video output to HDMI. The video can be changed remotely through a Samba share and a reboot of the Pi.

### Installing

Simple instructions to get everything up and running.

First, edit the unlockroot file, creating a different password for root.

```
# New password for root to use
rootpass="YourNewPassword"
```

Second, edit the automation file, changing the variables at the top to suit your use case. The variable names are fairly self-explanatory, and the examples try to help.

```
# Replace each variable with your desired names and passwords
computername="YourComputerName"
primaryuser="UserToReplaceUserPi"
primarypass="UserPisNewPassword"
secondaryuser="SecondaryUserThatCanAccessSambaShare"
secondaryuserdisplayname="SecondaryUser DisplayName"
secondarypass="SecondaryUserPassowrd"
group="UserGroupToBeCreatedForSambaPermissions"
```

The first line is for the new device name for the Pi. This also makes it easier to find on the network.

```
computername="YourComputerName"
```

The next two lines are for changing the username of the default user "Pi" and the associated password, helping your security in the process.

```
primaryuser="UserToReplaceUserPi"
primarypass="UserPisNewPassword"
```

Up next are settings for creating a new user that only has access to needed files for changing the looping video. The display name is used as a more user-friendly by the computer.

```
secondaryuser="SecondaryUserThatCanAccessSambaShare"
secondaryuserdisplayname="SecondaryUser DisplayName"
secondarypass="SecondaryUserPassowrd"
```

Lastly, we are adding a user group to handle all of these permissions. You can add more users to this group later, if you wish, just remember to also add them as Samba users.

```
group="UserGroupToBeCreatedForSambaPermissions"
```

With these variables set, we can now move on to the last part: editing wpa_supplicant.conf so we will automatically connect to our Wi-Fi network. This step can be skipped if you are using an ethernet connection instead. For that case, just do not copy this file over to the SD card.

```
# Replace SSID and PSK with the info needed for your network
network={
	ssid="YourWiFiName"
	psk="YourWiFiPassword"
}
```

Now that we have all of our files ready, we need to copy all of them into the boot folder of our Raspbian install.

Plug in the Pi with the files all loaded. The ssh and wpa_supplicant.conf files will be used and should disappear when you look again. ssh will enable ssh access, and the wpa_supplicant.conf file will store the settings for your Wi-Fi network so it can automatically connect each time. As said before, feel free to exclude this one if you are using an ethernet connection.

After a few minutes for the Pi to startup and connect to the network, you will need to SSH into the Pi. If it is a fresh install, you can ping the default name to find the IP address.

```
ping raspberrypi.local
```

Open up a terminal window (or use something like PuTTY for Windows), and connect to the Pi using the IP address you found.

```
ssh pi@192.168.1.3
```

The default password is

```
raspberry
```

Now we need to unlock the root user account. Now that we are logged in, run the unlockroot script.

```
/etc/boot/unlockroot
```

This will add a line to the sshd_config file which handles SSH permissions, connection parameters, and more. The added line allows the user root to log in through SSH. Once the script finishes, the Pi will reboot.

Once the Pi reboots, reconnect using SSH with the user root and the password you set.

```
ssh root@192.168.1.3
```

Run the automation script to finish the installation and configuration phase.

```
/etc/boot/automation
```

This script will update and upgrade your system, and then install the necessary packages. If you wish to avoid the upgrade for whatever reason, delete this from the scripts

```
&& apt -y upgrade
```

The two exfat packages are not required, but are helpful if you use an exfat formatted drive for large transfer.

The script will rename the default pi user account to the new name you specified and change the password. A secondary user account will be created, as well as a permissions group for accessing the folder share. Once the directories are created, the startup script for launching omxplayer with the video to be looped will be added. Samba will be setup and both users added to the list. The changes made to allow root to login will be reverted, the default keyboard will be set to US (makes it easier to type the $ in scripts), and the Pi will be renamed to the appropriate name. A reboot at the end finalizes the setup.

To complete the install, connect through Samba and replace the loopvideo.mp4 file with another video titled loopvideo.mp4. Power cycling the Pi will have the new video played at startup on an infinite loop.
