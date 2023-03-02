---
title: "Deploying Life Meets Code the Server"
date: 2017-08-26T16:07:04-06:00
draft: false
type: "post"
tags:
  - linux
  - centos
  - server
  - cloud
aliases:
  - /blog/deploying-life-meets-code-the-server
---

I want to keep a record of how I configured the server for this site and decided to share that info to help others and (hopefully) improve my setup. I've used [DigitalOcean](https://digitalocean.com/) in the past, but decided to try out [Vultr](https://vultr.com/) to host my site. I use CentOS for my servers so I used Vultr's CentOS 7 x64 image as the base. From my Vultr account dashboard, I chose the $2.50/month rig, uploaded my SSH key, set the hostname and label, and deployed the server.

First thing: login as root to create a new user:

```
$ ssh root@<SERVER_IP_ADDRESS>
#Create the new user
$ adduser <username>
#Set the user's password
$ passwd <username>
#Add user to the wheel group (for sudo access)
$ gpasswd -a <username> wheel
```

Exit the root's terminal session and add your SSH key to your new user.

```
#On local machine:
$ ssh-copy-id <username>@<SERVER_IP_ADDRESS>
```

You will be prompted for the password of the user you created on the server to install the key.

Now try to SSH in. You should not be prompted for a password.

```
$ ssh <username>@<SERVER_IP_ADDRESS>
```

Now that we have our new user, let's harden the SSH config.

```
$ sudo vi /etc/ssh/sshd_config
```

Look for these settings, remove any # and set the values as these:

```
PermitRootLogin no
PasswordAuthentication no
```

The settings are fairly self-explanatory, but I'll elaborate. Setting `PermitRootLogin` to `no` disables the ability to SSH into the server as root. Setting `PasswordAuthentication` to `no` only allows SSH logins with an SSH private key. You can still login with a password using the console of your provider. Reload the SSH daemon to pick up the changes.

```
$ systemctl reload sshd
```

Start a new terminal session and test if you can login as root. You should not be able to login. Now try logging in as your user. If you can't login at all through SSH, you can go in through console of your provider and revert the changes you made.

## SELinux

One thing I [learned about Vultr](https://hnm.run/post/vultr-centos-selinux/) is that they seem to remove SELinux during provisioning. To bring SELinux back:

```
#Install SELinux policies and dependicies
$ sudo yum install selinux-policy-targeted

#Set SELinux to enforcing
$ sudo setenforce 1

#Tell SELinux to relabel the filesystem at reboot
$ sudo touch /.autorelabel

#Restart the system
$ sudo reboot
```

After the system is back up, verify SELinux is enforcing.

```
$ sudo sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

## Update the system

Before starting installing software, let's ensure the base system is up-to-date

```
$ sudo yum update
$ sudo reboot
```

## Firewall

`firewalld` is installed by default, but verify that is it currently running.

```
sudo systemctl status firewalld
```

If the output contains `inactive (dead)`, then `firewalld` is not started.

```
 firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Sat 2017-08-26 15:30:50 UTC; 1s ago
     Docs: man:firewalld(1)
  Process: 466 ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 466 (code=exited, status=0/SUCCESS)
```

If necessary, start `firewalld`.

```
$ sudo systemctl start firewalld
$ sudo systemctl status firewalld

 firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2017-08-26 15:32:13 UTC; 8s ago
     Docs: man:firewalld(1)
 Main PID: 2056 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─2056 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
```

Since this server will host a website, we'll go ahead and allow ports 80 and 443 through the firewall.

```
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --reload
```

## Time zone and NTP

It's important to keep your server date/time in sync. As of CentOS 7, `chrony` is the default NTP service. Verify it's running:

```
$ sudo systemctl status chronyd

● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2017-08-21 02:30:21 UTC; 3min 41s ago
     Docs: man:chronyd(8)
           man:chrony.conf(5)
  Process: 542 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
  Process: 505 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 510 (chronyd)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/chronyd.service
           └─510 /usr/sbin/chronyd
```

Now, let's set the time zone of the server.

```
$ sudo timedatectl set-timezone region/timezone
# verify date/time
$ date
```

## Swap

Since the server instance does not have much memory, we'll create a swap file to help. In this case, we'll create a 2GB swap file.

```
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```

## Additional Hardening

### IP spoofing

To help against potential [IP address spoofing](https://en.wikipedia.org/wiki/IP_address_spoofing), you can make some [modifications to the /etc/host.conf file](http://www.tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap5sec39.html).

```
sudo vi /etc/host.conf
```

Edit the file to look like:

```
# multi on
order bind,hosts
nospoof on
```

Reboot the server to ensure changes take effect.

###  Validate with nmap

Install `nmap`.

```
sudo yum install nmap
```

Run `nmap`.

```
sudo nmap -v -sT localhost
```

Output should resemble:

```
Starting Nmap 6.40 ( http://nmap.org ) at 2017-01-07 11:31 CST
Initiating Connect Scan at 11:31
Scanning localhost (127.0.0.1) [1000 ports]
Discovered open port 25/tcp on 127.0.0.1
Discovered open port 22/tcp on 127.0.0.1
Completed Connect Scan at 11:31, 0.05s elapsed (1000 total ports)
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00039s latency).
rDNS record for 127.0.0.1: kamm-cloud
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.10 seconds
           Raw packets sent: 0 (0B) | Rcvd: 0 (0B)
```

Also run

```
sudo nmap -v -sS localhost
```

Output should resemble:

```
Starting Nmap 6.40 ( http://nmap.org ) at 2017-01-07 11:33 CST
Initiating SYN Stealth Scan at 11:33
Scanning localhost (127.0.0.1) [1000 ports]
Discovered open port 25/tcp on 127.0.0.1
adjust_timeouts2: packet supposedly had rtt of 2810839666214439 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 2810839666214439 microseconds.  Ignoring time.
Discovered open port 22/tcp on 127.0.0.1
adjust_timeouts2: packet supposedly had rtt of 2810839666214304 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 2810839666214304 microseconds.  Ignoring time.
Completed SYN Stealth Scan at 11:33, 0.02s elapsed (1000 total ports)
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
rDNS record for 127.0.0.1: kamm-cloud
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
           Raw packets sent: 1000 (44.000KB) | Rcvd: 2002 (84.088KB)
```

Once you've verified everything looks okay, you can remove nmap.

```
$ sudo yum remove nmap
```

### fail2ban

I used the DigitalOcean article, [How To Protect SSH With Fail2Ban on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-7), to get up and running. The basics are:

```
sudo yum install fail2ban
sudo systemctl enable fail2ban
sudo vi /etc/fail2ban/jail.local

---
[DEFAULT]
# Ban hosts for one hour:
bantime = 3600

# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = iptables-multiport

[sshd]
enabled = true
---

sudo systemctl start fail2ban
sudo fail2ban-client status
```

The status output should indidate that sshd is in a jail.

```
Status
|- Number of jail:  1
`- Jail list:   sshd
```

Now that the base system is ready, the next steps will be to get the web server up-and-running along with a deployment strategy for the site. To keep the posts short, I plan on writing a separate post for that.
