# secure-Linux Logging Lab

## Overview
A hands-on project to deepen my understanding of virtualization, networking, firewall management, server hardening, log forwarding, and automation — all built and maintained using Linux, rsyslog, ufw, and Ansible.

## Goals
Build and manage virtual machines using KVM/QEMU

Secure VMs using UFW, SSH key-based login, and subnet isolation

Create and manage a central logging server

Filter noisy logs with rsyslog

Automate tasks with Ansible

Enable remote connectivity using Tailscale VPN



## Current Progress

Set up multiple Ubuntu VMs

Configured SSH key-based login

Applied custom firewall rules

Created an isolated virtual subnet (virbr1)

Centralized logging from clients → server

Filtered noisy logs via rsyslog.d confs

Enabled secure remote access using Tailscale

Automate setup with Ansible (in progress)


## Firewalls (UFW)
Ubuntu’s Uncomplicated Firewall is used to secure each VM. The following rules allow essential traffic while blocking everything else by default:

sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 514/tcp  # For rsyslog logging


The following is the rules I settled on  

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443                        ALLOW       Anywhere                  
514/tcp                    ALLOW       Anywhere                  
22/tcp                     ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443 (v6)                   ALLOW       Anywhere (v6)             
514/tcp (v6)               ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6)   


Then I navigated to /etc/ssh/sshd_config 
''' #PasswordAuthentication no

SSH keys are used to authenticate from trusted machines (e.g., my MacBook).

Created a new user account, then set up key-based auth:

ssh-keygen
ssh-copy-id user@server

## Virtual Subnet & Networking

To simulate a safe, self-contained network:

A custom subnet (virbr1) was created

All VMs live on this subnet to isolate them from my home Wi-Fi

This enables tighter control and safer experimentation


I created a new vm so I can connect another machine to it and complete the following
1) needed to see if the subnet worked and had internet
2) send logging messages to the server and set up my logging messages to be forwarded to the port 514 through TCP.

## Centerallized Logging (rsyslog)

To forward all logs over TCP to the the logging server
I had to update the etc/rsyslog.conf

'''*.* @@<ipadddress>:514

Then I had to go to the ect/rsyslog.conf file on server and make sure tcp was enabled 


![image](https://github.com/user-attachments/assets/2f109d91-bec0-4c3d-980d-243046154261)

sudo -f tail -/var/log/syslog

I had to set up /etc/rsyslog.config to send logs from connected machines to their own dirctory
I did this by adding this to the file:


    # All logs (including remote) still go to syslog
    *.* /var/log/syslog


https://www.redhat.com/en/blog/log-aggregation-rsyslog (this webpage helped me figure this step out)


To make each sure logs of high priority were being addressed, I created configuration rules to direct those logs to a certain location. 

Within /etc/rsyslog.d the following directopries where written 

    10-remote.conf  20-ufw.conf  21-cloudinit.conf  30-sudo-usage.conf  40-ssh-fail-filter.conf  50-default.conf

These are included in the project files.


According to a stackoverflow user :

    /etc/rsyslog.d/ is just the directory where rsyslog's configuration files are kept. Rsyslog, just like the majority of the services nowadays, uses split configuration files (one file for each source) as opposed to a big configuration file containing all the sources. If you take a look into /etc/ you will find that many other services are organized in a similar way, for instance /etc/httpd/conf.d/.


This webpage helped me https://tp69.blog/2019/01/01/suppressing-messages-in-var-log-syslog/

##Conecting to my mac. 

I created a user for my macbook so I can work on my project on the go and see the logs
I created an account specifally for my macbook on my server

''' sudo useradd <name>
''' sudo passwrd <name>

Later on, I forgot I already made the user and used

''' getent passwrd <name>

this confirmed the user existed

to make it easy to sign on i created a public key on my mac and copied it on the correct directory

''' ~/.ssh/id_rsa.pub).

''' /home/<name>/.ssh/authorized_keys (on server)

I realized I wanted to be able to work on my project when I'm on the go, So I thought I needed to configure port forwarding. 

I tried to forward my Host machine's port 2222 to 22 on my VM 
 sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.100.1:22
 sudo iptables -t nat - A POSTROUTING -p tcp -d 192.168.100.1 --dport 22 -j MASQUERADE




Chain PREROUTING (policy ACCEPT 104 packets, 15144 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:2222 to:192.168.100.1:22

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 611 packets, 72339 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  6    --  *      *       0.0.0.0/0            192.168.100.1        tcp dpt:22



I failed to understand that my server and vm's are connected via my personal private virtual subnet. which uses a NAT network (virb1). This is isolated from my laptop which is connected to the home wifi So there was never any way for my macbook to get into my server. 

Even when I tried to ping it, there was no pakages reached. 

Then I decided to use Tailscale VPN to bridge network gaps. Each of my connected devices joins the same private network after i logged in via my GitHub account. These devices are given a stable 100.x.x.x IP so they can always communicate. 


![image](https://github.com/user-attachments/assets/22de410d-6ec8-4071-bc6e-fb93ef40980f)



## Cloud-init

To automate the process of automating vm deployment cloud-init weas used to confisgure them and forward its logs to the SIEM Server.

This file can be found in {ENTER_LOCATION}

#Useful Virsh Commands

to check vm status

    sudo virsh list --all

connect to the vm console

    sudo virsh console log-client-01

see the VM's IP(via DHCP)

    sudo virsh net-dhcp-leases nat0

See networks

    sudo virsh net-list --all
Stop/start vms

    sudo virsh start log-client-01
    sudo virsh shutdown log-client-01


#Log Rotation

I set up a basic log roatation file for safety

sudo vi /etc/logrotate.d/filtered-logs

/var/log/filtered/*.log {
    daily
    rotate 14
    compress
    missingok
    notifempty
    create 0640 syslog adm
    sharedscripts
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate || true
    endscript
}


## Connecting rsyslog to a SQLdatabase(PostgreSQL)

I used this as a reference/guide https://www.rsyslog.com/doc/tutorials/database.html

I set up a virtual python enviroment within my virtual ubuntu server, to ensure I didn't mess with my system's own version of python.

I had the following set up within my server to hold my UI path

I also had to set up the db which I named serverevents

log_dashboard ------- index.html
                ----- app.py
                ----- log_ingestor.py
                ----- templates ---


## User Interface 

My goal for this section is to make the index page and the log page attractive. I wanted more creative freedom so I used plain css/html.
I used graphs.css for more data visualization of my syslogs 


Here is a demo of my project:
                     





https://github.com/user-attachments/assets/c7c99122-dcb0-4d3d-96da-679493789802



I do plan on adding the follwoing 
    : some sort of alert system for certain cases and testing the security of it by using medusa or soemthing similar. 
    : fixing the user interface and making it more attractve
    : having the logs get some sort of tag via location to see the location of the threat
    - Authetication for the interface
    - Container the whole application using Docker
