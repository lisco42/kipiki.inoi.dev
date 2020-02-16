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
  * Change Keyboard Layout (changes the keyboard layout (In US the issue is the ``|`` sends a ~ with 105 intl, you need to change to 104))
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

## Building the local access point and bridge

Items covered in this section:

* Building an access point using the usb dongle

Great document from Arch linux on this process: https://wiki.archlinux.org/index.php/Software_access_point

Since the linked document covers a lot of checking for capability and link layer material which I suggest you go read, this section will just be setting up the raspian system for the wlan1 access point.

We will be creating an access point from the usb dongle to the physical network interface in this example.

**Installing packages**

First we will need to install the hostapd and bridge-utils packages: 

```bash
root@rpi:~# apt install hostapd bridge-utils
```

**Setting up hostapd.conf**

Now we can configure the access point, in this example I am making an open one as it is for free/open access to everyone at the place

```bash
root@rpi:~# vi /etc/hostapd/hostapd.conf
ssid=<your ap name>
interface=wlan1
bridge=br0
channel=6
hw_mode=g
auth_algs=1
wmm_enabled=0
```

I needed to also set up the defaults to use the proper config file (add to end of file)

```bash
root@rpi:~# vi /etc/default/hostapd
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

And have the service start up at boot

```bash
root@rpi:~# systemctl enable hostapd
```

**Setting up the bridge** 

This allows you to treat multiple interfaces as one (for routing, dhcp and the like) 

```bash
root@rpi:~# vi /etc/network/interfaces
...
auto eth0
iface eth0 inet manual
auto wlan1
iface wlan1 inet manual

auto br0
iface br0 inet static
	bridge_ports eth0 wlan1
		address 10.10.10.1
		broadcast 10.10.10.255
		netmask 255.255.255.0
		gateway 10.10.10.1
```

A quick reboot and you should end up with something like this: 

```bash
pi@rpi:~ $ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast master br0 state DOWN group default qlen 1000
    link/ether <mac addy> brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether <mac addy> brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.139/24 brd 10.0.0.255 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 fe80::ba27:ebff:fe08:be2/64 scope link 
       valid_lft forever preferred_lft forever
4: wlan1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master br0 state UP group default qlen 1000
    link/ether <mac addy> brd ff:ff:ff:ff:ff:ff
5: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether <mac addy> brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::821f:2ff:fe9a:a0b1/64 scope link 
       valid_lft forever preferred_lft forever
```

## Setting up DHCP/DNS

Items covered in this section:

* Setting up dnsmasq to broadcast DHCP over access point and physical lan
* Setting up local resolutions for DNS
* Setting up Stephen Black's Hostfile

**dnsmasq setup**

DNSMasq is the utility we will be using to both assign DHCP addresses and to relay DNS traffic.

Installing the package: 

```bash
root@rpi:~# apt install dnsmasq
```

Configuration:

* dhcp-authoritative: allows you to override anyone accidently broadcasting dhcp leases on your network. Be warey of connecting this box directly to existing networks as it will try to override the network's dhcp services and cause interruptions (the same as buying a linksys router and plugging the lan portion into a lan that already has dhcp/routing services!).
* dhcp-range: gives you the range where your clients will be given ips
* interface: the interface where dhcp assignments will be given out

```bash
# First move the old config out, it has a lot of useful information but we only want what we want so make it nice and short
root@rpi:~# mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
root@rpi:~# vi /etc/dnsmasq.conf
# Now set up your config
root@rpi:~# vi /etc/dnsmasq.conf
interface=br0
dhcp-range=10.10.10.100,10.10.10.230,1h
dhcp-authoritative
```

Now restart the service (its started automatically) and ensure that it starts at boot: 

```bash
root@rpi:~# systemctl restart dnsmasq; systemctl enable dnsmasq
```

**Local DNS resolutions and Stephen Black's Hostfile**

This section covers using Stephen Black's Hostfile. It is used for two things, and is configurable:
* Protect users from things like ads, tracking, malware, viruses
* Prevent users from going to nefarious sites.

As cool as the hosts file is, please take the following into consideration:
* If you want pure-internet access, do not perform this section. Things like ad services will not work with this enabled.
* I generally consider this as protect only, if you block stuff that users *want* to see, they will find a way around it.

Here's a small script I wrote to handle all this (run as root): 

```bash
#!/bin/bash
wget https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/fakenews/hosts -O /root/blackhost
cat /root/hosts.local > /root/hosts
cat /root/blackhost >> /root/hosts
cp /root/hosts /etc/hosts
chmod 544 /etc/hosts
/etc/init.d/dnsmasq restart
```

Files:
* /root/hosts.local: entries that are for local dns defns, will override everything else in file since it comes first, just copy your /etc/hosts here to begin with
* /root/blackhost: entries that are from the github of StephenBlack

Its probably best to have crontab run this @reboot so it can get a new hostlist from the github (with a 30 second delay to let the internet connection settle) (you can edit the crontab with crontab -e and will need to select a editor, I would suggest vi or nano) 

```bash
root@rpi:~# crontab -l
@reboot sleep 30; /root/hostsgen.sh
```

## Setting up routing, kernel options and restrictions

Covered in this section:

* Kernel options for ipv4 forwarding (not currently covered for ipv6, perhaps later the doc will be updated)
* IPTables Rules:
  * Basic routing
  * Basic restrictions

**Setting up ipv4 forwarding**

The kernel needs to be told to allow ipv4 ip forwarding for this to become a router. This can be done pretty simply:

Setting up for immediate use: 

```bash
root@rpi:~# echo 1 > /proc/sys/net/ipv4/ip_forward
root@rpi:~# cat /proc/sys/net/ipv4/ip_forward
1
```

Setting up to survive reboots:

```bash
root@rpi:~# vi /etc/sysctl.conf
# Uncomment the net.ipv4.ip_forward line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

**Iptables rules**

Setting up IPTables rules to be automatically loaded on boot, first make sure your not wiping out anything then apply the config: 

```bash
root@rpi:~# cat /etc/network/if-pre-up.d/iptables
cat: /etc/network/if-pre-up.d/iptables: No such file or directory
root@rpi:~# echo '#!/bin/sh
/sbin/iptables-restore < /etc/iptables.rules' > /etc/network/if-pre-up.d/iptables
root@rpi:~# chmod +x /etc/network/if-pre-up.d/iptables
```

Build your IPTables, first the routing tables (for letting things nat out of the network) 

```bash
root@rpi:~# iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
```

Then to restrict networks you may not want them getting to (this is optional/may change in your situation, dont just copy+pasta) 

```bash
root@rpi:~# iptables -A OUTPUT -j ACCEPT -d 10.1.10.1
root@rpi:~# iptables -A OUTPUT -j DROP -d 10.1.10.0/24
root@rpi:~# iptables -A INPUT -j ACCEPT -d 10.1.10.1
root@rpi:~# iptables -A INPUT -j REJECT -d 10.1.10.0/24
```

Now the rules are active, but you need to save them to have them be active on the next boot: 

```bash
root@rpi:~# iptables-save > /etc/iptables.rules
root@rpi:~# cat /etc/iptables.rules
# Generated by iptables-save v1.6.0 on Fri Sep  1 17:26:55 2017
*filter
:INPUT ACCEPT [197:27747]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [64:5768]
-A INPUT -d 10.1.10.1/32 -j ACCEPT
-A INPUT -d 10.1.10.0/24 -j REJECT --reject-with icmp-port-unreachable
-A OUTPUT -d 10.1.10.1/32 -j ACCEPT
-A OUTPUT -d 10.1.10.0/24 -j DROP
COMMIT
# Completed on Fri Sep  1 17:26:55 2017
# Generated by iptables-save v1.6.0 on Fri Sep  1 17:26:55 2017
*nat
:PREROUTING ACCEPT [8:1108]
:INPUT ACCEPT [8:1108]
:OUTPUT ACCEPT [2:180]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o wlan0 -j MASQUERADE
COMMIT
# Completed on Fri Sep  1 17:26:55 2017
```

After reboot you can check to make sure your rules are active: 

