# secure-home-lab

## Overview
Project to deepen understanding of virtualization,networks, server hardening, firewall management, logging, and automation. KVM server, SSH, firewall polices, Ansible

## Goals
Learn virtualization. In this project I used (KVM)
Explore networking and firewall configuration
Implement logging and automation

## Progress
VM set up
SSH set up
Key set up
Basic Firewall set up


## Firewalls (UFW)
To set up my firewalls, the ubuntu uses UFW (uncomplicated firewalls).
I want the server to allow ssh, http, https connections. I used the follwoing commands to enable ufw, allow the various permissions I wanted.

''' sudo ufw enable
was used to enable the firewall

''' sudo ufw allow OpenSSH
Used to allow sercure connections so I can log into the server from various machines

''' sudo ufw allow http
''' sudo ufw allow https
Used to allow http and https connections

The following is the rules I settled on  

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp (OpenSSH)           ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
443                        ALLOW IN    Anywhere                  
514/tcp                    ALLOW IN    Anywhere                  
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443 (v6)                   ALLOW IN    Anywhere (v6)             
514/tcp (v6)               ALLOW IN    Anywhere (v6)    

Then I navigated to /etc/ssh/sshd_config 
''' #PasswordAuthentication no
checking with my macbook, I cannot SSH in. I need the key on my machine in order to login

## Network

Created a subnet (yellowsubmarine) so I can have a more unified way to connect all my machines. Most importantly, it is safer than using my local internet and masks it in some way

## New V< (ubuntu Workstation)

I created a new vm so I can connect another machine to it and complete the following
1) needed to see if the subnet worked and had internet
2) send logging messages to the server and set up my logging messages to be forwarded to the port 514 through TCP.

The subnet worked.

I had to update the etc/rsyslog.conf so the system can forward all logs to the server via TCP through port 514

'''*.* @@<ipadddress>:514

Then I had to go to the ect/rsyslog.conf file on server and make sure tcp was enabled 


![image](https://github.com/user-attachments/assets/2f109d91-bec0-4c3d-980d-243046154261)

sudo -f tail -/var/log/syslog

I had to set up /etc/rsyslog.config to send logs from connected machines to their own dirctory
I did this by adding this to the file:
''' $template RemoteLogs,"/var/log/%HOSTNAME%/remote_logs.log"

local0.* action(type="omfile" dynaFile="RemoteLogs")

$IncludeConfig /etc/rsyslog.d/*.conf

https://www.redhat.com/en/blog/log-aggregation-rsyslog (this webpage helped me figure this step out)

When the logs came it, it was a lot of junk inforamtion. Just as updates taking place and cron job. An overwhelming amount of systemd messages of  processes starting and finsihing.
I had to edit my conf file, so that noise could be surpressed. 


#
# Include all config files in /etc/rsyslog.d/
#
$template RemoteLogs,"/var/log/%HOSTNAME%/remote_logs.log"
local0.* action(type="omfile" dynaFile="RemoteLogs")

# filitering out the noise 

if($programname == 'gnome-shell') then stop
if($programname == 'systemd') then stop
if($programname == 'rsyslogd') then stop
if($programname == 'rtkit-daemon') then stop
if($programname == 'update-notifier') then stop
if($programname == 'CRON') then stop

This webpage helped me https://tp69.blog/2019/01/01/suppressing-messages-in-var-log-syslog/



