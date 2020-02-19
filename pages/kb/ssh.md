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

```bash
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

```bash
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

```bash
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

```bash
user01@lug01:~$ ssh-keygen -f "/home/user01/.ssh/known_hosts" -R "172.16.54.2"
# Host 172.16.54.2 found: line 1
/home/user01/.ssh/known_hosts updated.
Original contents retained as /home/user01/.ssh/known_hosts.old
```

* This is the key from the client side (lug01)

```bash
user01@lug01:~$ cat .ssh/known_hosts
|1|gJ8Ievpj9e7A4C/MCV84j5FPrgM=|bgP//KsWhRs/f6VwmUbtE75Mzho= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPeWq64+GAuUV1rfRFXkDBmPDhO02m1f3qAGPvitmo4DKaN2G2BZ10AEfUWxestAgPsjt4nWgIHkMnyg29hDhS0=
```

* Here is the key from the server side (lug02) - public then private (this should go without saying, but private key should never be shared, this is an example machine)

```bash
root@lug02:/etc/ssh# cat ssh_host_ecdsa_key.pub
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPeWq64+GAuUV1rfRFXkDBmPDhO02m1f3qAGPvitmo4DKaN2G2BZ10AEfUWxestAgPsjt4nWgIHkMnyg29hDhS0= root@lug02
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

```bash
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

```bash
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

```bash
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

## Securing

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
# Alternate port setup (protects against mindless bots, but not usually real attacks)
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
# Restricting the ciphers and macs that ssh will support prevents things like attacks that make the client think the server supports lower, broken encryption types, however many of these can break compatibility with certain types of clients, like putty
Ciphers aes256-ctr
MACs hmac-sha2-512
```

Login hammering mitigation: 
* fail2ban

```bash
# Fail2ban stops server hammering by watching the authlogs and if triggered, will initiate mitigations based on the linux firewall system.  The default debian config for jails.conf includes ssh (below) but if you want more restrictive settings you can modify them.

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
### Dynamic Socks Proxy
### Static Port Forwarding

## Jumphost

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