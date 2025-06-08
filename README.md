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
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443 (v6)                   ALLOW IN    Anywhere (v6)             


Then I navigated to /etc/ssh/sshd_config 
''' #PasswordAuthentication no
checking with my macbook, I cannot SSH in. I need the key on my machine in order to login

## Network

Decided I wanted a subnet so I created one
yellowsubmarine

## New V< (ubuntu Workstation)




