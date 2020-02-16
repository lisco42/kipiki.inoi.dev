# PortaPi Router

A portable router/server based on raspberry pi

## Synopsis

Building a very small portable lan for groups like the 2600 and LUG where you want shared resources without bringing in the big machines/routers/ect but still providing a useful environment and locked down enough to prevent people from exploiting local resources.

This document's goal is to put together a full knowledge dump from beginning to end of the process of making one of these routers. It tries not to assume any knowledge, but light knowledge in how networks work along with linux (IE the difference between an external and internal interfaces) are very helpful. 

**Use Cases**

* Private network in a public place
* Provides firewall to your devices, isolating them from the open network and potential attack
* The ability (not covered here yet) to tunnel all of your traffic over an encrypted vpn keeping you from having to configure every device with vpn
* When at hotel that only provides access for one or two devices, and setting up devices like a chromecast are not feasable when every device is required to login
* Having local services for you and your friends to use all on a private network

**Points to accomplish with build**

* Have one to two pieces, small enough to comfortably fit in small backpack compartment or pocket
* No interaction required at meeting to have it up and running, steps: 1> apply power, 2> drink beer
* Automatically set up on wireless - pre-configured to 

**Hardware Used**

* Raspberry Pi 3 B+ Wireless -- alternate Raspberry Pi Zero W
* Sandisk Ultra 32GB - I suggest using Sandisk Ultra or Extreme cards as they have error correcting, cheap SD cards usually die in short order being used as root for a computer.
* Battery - Must be able to supply 2+Amps over 5V, short cables preferred to reduce any voltage loss.
  * Tested batteries:
    * Anker 20000 mAh battery that can put out 4A - works for pi3 + switch
    * Poseidon 10000 mAh battery 2.4 A and 1A ports - works for pi3 (on 2.4 port) and should work for switch (1A port)
    * Anker 5000 mAh battery that has one port but 2A output - pi3 is able to boot but under any load as router it crashes
  * More reading on power consumption: https://www.pidramble.com/wiki/benchmarks/power-consumption
  * Testing with just the pi on the 20000mah battery yeilded over 2 days of pi at idle, which will be significantly less with the switch and higher load, hoping for around 6+ hours.
* Wireless nic for making access point, tested with:
  * Edimax nano EW-7811Un (Realtek RTL8188CUS)
  * TP-Link TL-WN772N (more powerful for longer range access point)
* Switch that can run off 5v - For me a Trendnet TEG-S5g then using a direct usb -> 2.1mm barrel connector to power
* Short Ethernet cable
* Short USB cable (Pi Power)
* Short USB -> 2.1mm jack (Switch Power)

**Software Used**

* Raspbian lite image (debian stretch) - [Raspbian Downloads page](https://www.raspberrypi.org/downloads/raspbian/)

**Random Notes**

* I use both sudo and straight up root to do many things in this document, I try to keep consistent within a section, but wanted to show off both methods. I prefer to go to the root shell when doing many actions opposed to doing sudo for each action. There are many people in both camps on preference. That being said, do not just run everything as root, only what you must. This document is mostly about configuration in the operating system, so most of it is done by root. 

## Buildout Process
### Image Deployment

First we are going to download, extract then stick the image on our microsd card

I will be using a linux machine to perform these initial imaging steps. 
* If you are using a windows machine: [Raspberry Pi documentation for imaging using windows](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)
* If you are using an OSX (Mac) machine: [Raspberry Pi documentation for imaging using mac](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)

Deploying the image:
* Download whatever linux your using (I'm using raspbian / debian stretch in this case) using whatever method you like, I used the torrent via deluge
* Put your microsd into a reader, and insert into your linux box
* Find what your device got named, on a single disk modern linux system it is probably /dev/sdb
* Make sure you do not have data on the microsd that you want to preserve, the following actions will erase the microsd. My example is for /dev/sdb, ensure you use the proper device for your microsd, I am not responsible for destroyed data.

**The following commands are run as root:**

```bash
# fdisk -l /dev/sdb
Disk /dev/sdb: 29.7 GiB, 31914983424 bytes, 62333952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8c9f67fa

Device     Boot Start      End  Sectors  Size Id Type
/dev/sdb1        2048 62333951 62331904 29.7G  b W95 FAT32
# time dd if=2017-08-16-raspbian-stretch-lite.img of=/dev/<the sdcard such as sdb> bs=4M
442+1 records in
442+1 records out
1854418944 bytes (1.9 GB, 1.7 GiB) copied, 199.563 s, 9.3 MB/s

real	3m19.565s
user	0m0.000s
sys	0m1.727s
# fdisk -l /dev/sdb
Disk /dev/sdb: 29.7 GiB, 31914983424 bytes, 62333952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xee397c53

Device     Boot Start     End Sectors  Size Id Type
/dev/sdb1        8192   93813   85622 41.8M  c W95 FAT32 (LBA)
/dev/sdb2       94208 3621911 3527704  1.7G 83 Linux
# sync
```

## Setting up Raspberry Pi

Items covered in this section:

* Hooking up pi
* Set user password
* Set hostname
* Start SSHD
* Update system

**Hooking up pi**

After the image is deployed to the microsd card, put it into the pi, hook up the pi to a monitor, keyboard, wired internet connection

Power the pi on (connect it to a 2+Amp usb power source), it will automatically resize its second partition (root) to fill the sd card then reboot.

Log into the pi, default credentials:
* User: pi Pass: raspberry

**Initial setup of raspbian**

After logging into the pi as the pi user, configure the os: 

```bash
pi@raspberrypi:~ $ sudo raspi-config
```

To navigate: use the arrow keys and tab What to change using the menu (and what each option does):

* Localization Options
 * Change Locale (This changes the default/supported languages of the system, UK by default for raspbian)
  * Remove en_GB.UTF-8 UTF-8
  * Add en_US.UTF-8 UTF-8 (or whatever other lang/countries you want)
 * Change Timezone (setting up your local timezone)
  * For me it was US -> Eastern
 * Change Keyboard Layout (changes the keyboard layout (In US the issue is the | sends a ~ with 105 intl, you need to change to 104))
  * Change Generic 105-key (intl) PC to Generic 104-key PC
  * Keyboard Layout: (default English(UK), choose other)
  * English (US)
  * English (US)
  * The default for the keyboard layout
  * No compose key
 * Change Wi-Fi Country (this affects what channels you are allowed to use)
  * Select US United States
* Advanced Options
 * Memory split
  * I chose 16 here since we are going to be running this headless most of the time, this gives the OS the most memory and provides better performance for our purposes
* Finish

Now reboot the machine, then when it comes back up, set your password and hostname.
The reason we do the localization first, is when your putting in your password/hostname, the keyboard layout may be different, and if you change it with the wrong layout, then change the layout, you may not be able to log in (not from personal experience or anything).

* If you have locked yourself out of the pi:
 * Shut off the pi
 * Pull the microsd card, load into your linux box
 * mount partition 2 (IE mount /dev/sdb2 /mnt)
 * edit the /mnt/etc/shadow file, find the 'pi' entry, remove the second field (fields are denoted with a :)
 * unmount the media, put back into the pi, boot the pi
 * pi user will no longer have a password, set a new password using `passwd` or the method below.

 ```bash
 pi@raspberrypi:~ $ sudo raspi-config
 ```

* User password
 * This simply runs (as root) `passwd pi`
* Hostname
 * This changes the /etc/hostname and /etc/hosts entries to rename the machine

**Setting up sshd**

SSH (daemon) allows you to ssh to the new router and not have to be physically attached. It is installed by default in raspbian.

Steps in setting up sshd to automatically come up at boot:

```bash
# become root using sudo
pi@rpi:~ $ sudo -i

# Create a root password (useful if your user gets locked out and you need to log in with superuser directly)
root@rpi:~# passwd  

# Edit /etc/ssh/sshd_config (I use vi, others might use vim or nano [easiest].  If you dont know what vi is and how to use it, use nano as in the example below) Note: This is not strictly necessary, I do it because I never want to log in using root user remotely, and I do set up a password for root account
root@rpi:~# nano /etc/ssh/sshd_config 
## Inside sshd_config: add the line: PermitRootLogin no
## then write the file and exit the editor

# Set the ssh daemon to start at boot and start it
root@rpi:~# systemctl enable ssh; systemctl start ssh

# Determine your IP, and ssh (if you want) to the pi from another box
## my ethernet was an unfortunate mix of enx and the full mac address, will address this soon
root@rpi:~# /sbin/ifconfig
```

Then on your other machine, ssh to the pi: 

```bash
ssh pi@<ip address>
```

If you are old-hat like me, and want the original eth0/wlan0/ect instead of enx<16digitmac> (predictable but ugly as f), just do this: 

```bash
root@rpi:~# ln -s /dev/null /etc/systemd/network/99-default.link
root@rpi:~# reboot
```

**Update the system**

You should do this periodically, preferably every time before you take the pi out to an event to make sure you have the most modern patches for security 

```bash
root@rpi:~# apt update
root@rpi:~# apt upgrade
```

## Setting up WPASupplicant

Items covered in this section:

* General wireless tools
* Having the internal wireless card connect to a building wireless access point automatically using wpa_supplicant

**Packages you need**

wpasupplicant

wireless-tools



**Checking out what is in the area**

You can run the following to get a list of the access points in the area: 

```bash
root@rpi:~# iwlist wlan0 scan | grep ESSID
                    ESSID:"<ap name 0>"
                    ESSID:"<ap name 1>"
                    ESSID:"<ap name 2>"
                    ...
```

**Configuration of wpa_supplicant.conf**

First we will create the configuration for the password 

```bash
root@rpi:~# wpa_passphrase <access point name here>
# reading passphrase from stdin
<password here, it will appear but not be in your history>
network={
	ssid="<AP Name>"
	#psk="<password cleartext, remove this line>"
	psk=<encrypted password>
}
```

Copy that output, and paste it (using an editor) into the /etc/wpa_supplicant/wpasupplicant.conf file (you could also generate and use redirect (>>) to put the output into the file directly). You should end up with something like this, you can put as many in here as you want. 

```bash
root@rpi:~# vi /etc/wpa_supplicant/wpasupplicant.conf
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
	ssid="<AP Name>"
	psk=<encrypted password>
}
```

**Setup debian interfaces file**

Now you can set up the interfaces file /etc/network/interfaces. Add the following to your configuration. 

```bash
root@rpi:~# vi /etc/network/interfaces
allow-hotplug wlan0
iface wlan0 inet dhcp
	wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

Restart the system and it should come up on the wlan0 only (your ip will change to ssh to, you should be able to disconnect the physical lan after this point as to not have the following steps mess with your local network.)

Note on restarting the system: I wanted to just do an `ifup wlan0` but the if-pre-up.d/wpasupplicant script wasnt having that. Checked the wpasupplicant service, and it was up, but ended up having to reboot to have the configuration work. A restart of the network substructure would have probably accomplished the same, but since I edited the interfaces file, the PFM that systemd is doing to the network stopped working with eth0 which we will be addressing during the dns/dhcp section. 

