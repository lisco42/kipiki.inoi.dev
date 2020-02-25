# SSH - Secure Shell

## Overview
Secure shell allows for secure remote access to machines.  
* It is available in Linux, BSD, OSX and Windows and is very useful for various operations.
  * Secure remote shell access
  * Remote file system access
  * Socks 4/5 proxy for dynamic proxying traffic
  * Tunneling port access to remote hosts
  * X11 Forwarding to do remote management of GUI resources (linux)

## A common set of traffic flows between services visualized:
![SSH Flow](/kb_images/ssh_security_flow.jpg)

## Keys
### Server identification keys
Server identification keys are important for preventing man in the middle attacks.
* On first connection you will be asked if you trust the remote machine's key.
* Subsuquent connections (from the same client/server) will authenticate this automatically
* Server keys can be transferred (with root access) between servers, if for instance dealing with a round robin ssh bastion cluster, or rebuilding a server

* Example initial connection, followed by a subsuquent connection showing the key did not change:

```
user01@lug01:~$ ssh 172.16.54.2
The authenticity of host '172.16.54.2 (172.16.54.2)' can't be established.
ECDSA key fingerprint is SHA256:VjBWSuFHiK2AHtld34YAnfukXaX4twQgwqCoKV0siUo.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.54.2' (ECDSA) to the list of known hosts.
user01@172.16.54.2's password:
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
user01@lug02:~$ logout
Connection to 172.16.54.2 closed.
user01@lug01:~$ ssh 172.16.54.2
user01@172.16.54.2's password:
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Feb 19 12:36:15 2020 from 172.16.54.1
user01@lug02:~$
```

* If you have not rebuilt the server (and not transferred the key), and are using a client that has the key already in place, and see the following error... **BEWARE**
  * For this example I've removed the keys on lug02, and regenerated them as such:

```
root@lug02:/etc/ssh# rm ssh_host_*
root@lug02:/etc/ssh# ls
moduli  ssh_config  sshd_config
root@lug02:/etc/ssh# dpkg-reconfigure openssh-server
Creating SSH2 RSA key; this may take some time ...
2048 SHA256:k83S3dAtwsPX/q5EnKZjULNYOPWq1EqhrFk5VfWDahs root@lug02 (RSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:C1MPpJFvPMBffxs56bWa96lopnxvKH9tc5Cl5Lx1GUY root@lug02 (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:2M+TmQuKrc6C5WNg/yVsQLLnGzXBHsSLj7Gsr7cWqqs root@lug02 (ED25519)
```

* On reconnecting, the client errors with the following and should be heeded:

```
user01@lug01:~$ ssh 172.16.54.2
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:C1MPpJFvPMBffxs56bWa96lopnxvKH9tc5Cl5Lx1GUY.
Please contact your system administrator.
Add correct host key in /home/user01/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/user01/.ssh/known_hosts:1
  remove with:
  ssh-keygen -f "/home/user01/.ssh/known_hosts" -R "172.16.54.2"
ECDSA host key for 172.16.54.2 has changed and you have requested strict checking.
Host key verification failed.
```

* Clearing it (from like a host rebuild or something) is pretty simple, just follow the directions or remove the line from .ssh/known_hosts:

```
user01@lug01:~$ ssh-keygen -f "/home/user01/.ssh/known_hosts" -R "172.16.54.2"
# Host 172.16.54.2 found: line 1
/home/user01/.ssh/known_hosts updated.
Original contents retained as /home/user01/.ssh/known_hosts.old
```

* This is the key from the client side (lug01)

```
user01@lug01:~$ cat .ssh/known_hosts
|1|gJ8Ievpj9e7A4C/MCV84j5FPrgM=|bgP//KsWhRs/f6VwmUbtE75Mzho= ecdsa-sha2-nistp256 
AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPeWq64+GAuUV1rfRFXkDBmPDhO0
2m1f3qAGPvitmo4DKaN2G2BZ10AEfUWxestAgPsjt4nWgIHkMnyg29hDhS0=
```

* Here is the key from the server side (lug02) - public then private (this should go without saying, but private key should never be shared, this is an example machine)

```
root@lug02:/etc/ssh# cat ssh_host_ecdsa_key.pub
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPeWq64
+GAuUV1rfRFXkDBmPDhO02m1f3qAGPvitmo4DKaN2G2BZ10AEfUWxestAgPsjt4nWgIHkMnyg29hDhS0= root@lug02
root@lug02:/etc/ssh# cat ssh_host_ecdsa_key
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQT3lquuPhgLlFda30RV5AwZjw4TtNpt
X96gBj74rZqOAymjdhtgWddABH1FsXrLQID7I7eJ1oCB5DJ8oNvYQ4UtAAAAqIy0aBuMtG
gbAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPeWq64+GAuUV1rf
RFXkDBmPDhO02m1f3qAGPvitmo4DKaN2G2BZ10AEfUWxestAgPsjt4nWgIHkMnyg29hDhS
0AAAAgXe8LSEdnowObdCxxXlbi2klU07rPhjhMoghvB1WO+SUAAAAKcm9vdEBsdWcwMgEC
AwQFBg==
-----END OPENSSH PRIVATE KEY-----
```

### User identification keys
Building keys is a great way to stay secure, and not have to worry about someone getting into your system with compromised username and password combos (more on configuring your server in this way later)
* First you need to generate your ssh key on a client, this is transferrable, should have a password, and the private key should be kept... private:

```
user01@lug01:~$ ssh-keygen -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user01/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user01/.ssh/id_rsa.
Your public key has been saved in /home/user01/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:t5BqULaAVF73zydITcZaP1T4+QaDCRSawfMQcYsHl/Q user01@lug01
The key's randomart image is:
+---[RSA 4096]----+
|  ... ..*=*+o  o.|
| . o . .+Xo*o o  |
|  . o o ++=+E= ..|
|     + . +o+o =..|
|    . . S o + .+.|
|     . . o . o  o|
|      o   .    . |
|     .           |
|                 |
+----[SHA256]-----+
```

* After you have created your client key, you can then push the public part of that key to servers you have access to and enjoy more-secure access
* This action relies on your having password access to the remote server, using a key helps security in two ways
  * A key will protect your password from MITM attacks since it is based on cryptographic signing
  * If the server is, after you place the key on it, reconfigured to only accept key access, no password can be used to log in

* Pushing the key to a remote server and using it (this key has no password, as an example)

```
user01@lug01:~$ ssh-copy-id 172.16.54.2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user01/.ssh/id_rsa.pub"
The authenticity of host '172.16.54.2 (172.16.54.2)' can't be established.
ECDSA key fingerprint is SHA256:C1MPpJFvPMBffxs56bWa96lopnxvKH9tc5Cl5Lx1GUY.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user01@172.16.54.2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '172.16.54.2'"
and check to make sure that only the key(s) you wanted were added.

user01@lug01:~$ ssh 172.16.54.2
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Feb 19 13:47:40 2020 from 172.16.54.1
user01@lug02:~$
```

## MOTD
The 'Message of the Day' or MOTD is held at /etc/motd, it with /etc/issue are generally configured to be displayed to clients on login they have the following uses:
* Can provide up-to-date information to users of the system
* Can provide legal information and warnings
* Can put in fun little things to entertain yourself

* Putting in a custom MOTD is pretty simple, as root just modify /etc/motd or /etc/issue and you'll get your custom info on login, if we wanted to provide the user with a fun greeting on login, you could do this:

```
user01@lug02:~$ cowsay "Hello user, have a nice day!"
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

# Then as root, using your favorite editor, just add the material to /etc/motd
user01@lug02:~$ cat /etc/motd
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

# And now you'll have a cool login
user01@lug01:~$ ssh 172.16.54.2
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 14:07:31 2020 from 172.16.54.1
user01@lug02:~$
```

## Securing SSH_Daemon

A few config options:
* Disabling root login
* Alternate ports
* Limiting listening IPs
* Use Protocol 2 only
* Disable password login
* Use strict mode
* Ciphers

```bash
# Disabling root login, always a good measure, only allows for root to log on physically to the box
PermitRootLogin no
# Alternate port setup (protects against mindless bots, but not usually real attacks), you can even set it up for 
# multi-port so you can use one port internally and others externally
Port 2022
# If the box has internal and external ip's, you can make it only listen on one (like internal)
ListenAddress 172.16.54.2
# Restrict to only protocol 2
Protocol 2
# Disabling password login only allows for keys, more secure but more restrictive
PasswordAuthentication no
PubkeyAuthentication yes
# Strict mode makes ssh angry at you if you do things like have bad permissions or ownerships
StrictMode yes
# Restricting the ciphers and macs that ssh will support prevents things 
# like attacks that make the client think the server supports lower, broken encryption types, 
# however many of these can break compatibility with certain types of clients, like putty
Ciphers aes256-ctr
MACs hmac-sha2-512
```

Login hammering mitigation: 
* fail2ban

```bash
# Fail2ban stops server hammering by watching the authlogs and if triggered, 
# will initiate mitigations based on the linux firewall system.  The default debian 
# config for jails.conf includes ssh (below) but if you want more restrictive settings you can modify them.

[sshd]
# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

MultiFactor Authentication (links below):
* yubikey
* google 2fa

PAM Modules:
* allows for authentication manipulation including LDAP/2FA/other mitigations

## Dynamic Proxy and Port Forwarding
* Dynamic Socks Proxy

Dynamic Socks proxying is great!  You open a single tunnel, point your socks-capable product like firefox at it, and boom, your traffic pops out the endpoint, even if thats a few endpoints down because of jumphost

The -D flag tells ssh to open a socks 4 or 5 tunnel to the endpoint

```
user01@lug01:~$ ssh -D 8080 172.16.54.2
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 17:02:37 2020 from 172.16.54.1
user01@lug02:~$
```

And you can now see that you have a local port ready for socks clients:

```
user01@lug01:~$ netstat -anl | grep 8080
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN
tcp6       0      0 ::1:8080                :::*                    LISTEN
```

* Static Port Forwarding

Forwarding ports is very powerful and useful, have an RDP machine you need to get to but only have ssh access to an external server for safety, you can do that, have insecure webservers in a secure network, you can access them.  This is a 1:1 relationship whereas the above socks dynamic proxying is more robust, but less services can use it (they need to support socks)

the -L flag tells ssh to open a local socket to a remote socket thus creating a 'tunnel'

Setting up the initial connection, for instance this would forward traffic from 8443 on your local machine (lug01) to the remote host 8.8.8.8 (google dns) port 443

```
user01@lug01:~$ ssh -L 8443:8.8.8.8:443 172.16.54.2
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 16:54:57 2020 from 172.16.54.1
user01@lug02:~$
```

And showing us that we get to google dns webserver when we do a curl against it:

```
user01@lug01:~$ curl -k https://localhost:8443
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

* X11 Forwarding

-X and -Y enable ssh forwarding for your connection

![X11 Forwarding](/kb_images/x11_forwarding.jpg)

## Jumphost

Jumphost is a fantastic way to get into networks that have deep alignments.  Paired with keys and master connections you can be pretty safe by hopping in one or multiple layers of networks.

The -J option is for jumphost or proxyjump, and is a shortcut to an older, more difficult method of doing proxy jumps.  You can add one or more hosts by doing a comma seperated list.

Here is an example where (only using two servers but you get it):

The user is going from: user01 (local, lug01) -> user01 (remote, lug02) -> user01 (remote, lug01) -> jsmith (remote, lug02)

```
user01@lug01:~$ ssh -J 172.16.54.2,172.16.54.1 jsmith@172.16.54.2
user01@172.16.54.1's password:
jsmith@172.16.54.2's password:
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 17:15:46 2020 from 172.16.54.1
```

## User and port setup

* User switching

You can be whoever you want to be, within reason, if you're jack on your system, but your jsmith on the remote system, you can change your user to match

To do this, you can use user@server or the -l (lowercase L) flag and the remote name

```
user01@lug01:~$ ssh jsmith@172.16.54.2
jsmith@172.16.54.2's password:
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
$
Connection to 172.16.54.2 closed.
user01@lug01:~$ ssh 172.16.54.2 -l jsmith
jsmith@172.16.54.2's password:
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 17:15:17 2020 from 172.16.54.1
```

* Port switching

If you are using a different port then standard, you will have to tell the ssh client what port you are trying to reach, you can do that with the -p flag like so:

```
user01@lug01:~$ ssh -p 2022 172.16.54.2
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64
 ______________________________
< Hello user, have a nice day! >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Last login: Wed Feb 19 17:06:26 2020 from 172.16.54.1
```

## SSH client config

These are a few choice options that help me work more efficiently that you guys may enjoy

* Control Master connections

Master connections are fantastic ways of utilizing a secured infrastructure where you need to authenticate to a external protection server like a bastion, then spawn connections from there.  If you are not fond of logging in a thousand times to get to different servers, master connections are great!  Examples of it working will be on the SSH Client Config section as that is where I ususally use it, but the -M flag can be used on the command line to invoke it:

![SSH Master Connections](/kb_images/ControlMasterSSH.jpg)

* ServerAliveInterval

ServerAliveInterval prevents the connection from dying due to being idle.  

If you have a system that is set up to kill dead connections, this is a way to make sure the client keeps in touch with the server while your machine is still active.

* Server naming

You can name servers in your ssh config, even if they do not have dns for ease of access

* User, port, key and other changes

You can put pretty much everything you do on the command line, into the ssh config file, and if you need to connect to the same servers time and again, its a great reasource for all of the settings

* Example of a system set up with the above fun changes

```
user01@lug01:~$ cat ~/.ssh/config
Host *
  ServerAliveInterval 60

Host lug02
  User jsmith
  HostName 172.16.54.2
  Port 2022
  ControlMaster auto
  ControlPath ~/.ssh/%C
  DynamicForward 8080
```

And using it lets us log in to lug02, automatically as jsmith user, on port 2022, creating a socks tunnel on 8080, and using master so the second connection will just ride the first using multiplexing:

```
user01@lug01:~$ ssh lug02                                                       
jsmith@172.16.54.2's password:                                                  
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64          
 ______________________________                                                 
< Hello user, have a nice day! >                                                
 ------------------------------                                                 
        \   ^__^                                                                
         \  (oo)\_______                                                        
            (__)\       )\/\                                                    
                ||----w |                                                       
                ||     ||                                                       
Last login: Wed Feb 19 17:59:34 2020 from 172.16.54.1                           
$                                                                               

## and on the second connection:
user01@lug01:~$ ssh lug02
Last login: Wed Feb 19 18:02:30 2020 from 172.16.54.1
$ uname -a
Linux lug02 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64 GNU/Linux
$ id
uid=1001(jsmith) gid=1001(jsmith) groups=1001(jsmith)
```

## Links
* YubiKey 2fa:
  * https://developers.yubico.com/PGP/SSH_authentication/
  * https://github.com/drduh/YubiKey-Guide
* Google E2FA:
  * 
* General SSH Info: 
  * https://stribika.github.io/2015/01/04/secure-secure-shell.html
  * https://github.com/stribika/stribika.github.io/wiki/Secure-Secure-Shell
  * https://www.ssh.com/ssh/sshd_config/

## TODO: (things to be added post-presentation)
Somehow I forgot some of these simple things, some are more complex but forgotten none-the-less, will be removed from this list as they are added:
* scp
* sftp
* piping
* tar over ssh
* rsync over ssh
* dd over ssh
* gzipping via pipes
* connect proxy
* sshuttle (new to me, very cool) - basically vpn over ssh, shown by one of the jaxlug members to me.