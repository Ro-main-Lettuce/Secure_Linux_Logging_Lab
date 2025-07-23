#  Secure Linux Logging Lab

A hands-on project to deepen my understanding of:

    Virtualization (KVM/QEMU)

    Networking and subnet isolation

    Firewall configuration (UFW)

    Server hardening and secure access

    Centralized log aggregation (rsyslog)

    Automation (Ansible, cloud-init)

    Remote connectivity (Tailscale VPN)

    Log analysis and visualization (Flask dashboard)

    
## Goals

     Build and manage virtual machines using KVM/QEMU
    Secure VMs using UFW, SSH key-based login, and isolated subnets
    Create a central logging server
    Filter and categorize logs with rsyslog
    Automate deployment and configuration with Ansible
    Enable remote connectivity using Tailscale VPN



##  Current Progress

    ✅ Set up multiple Ubuntu VMs

    ✅ Configured SSH key-based login

    ✅ Applied custom UFW firewall rules

    ✅ Created an isolated virtual subnet (virbr1)

    ✅ Centralized logging from clients to the server

    ✅ Filtered logs via rsyslog.d configurations

    ✅ Enabled secure remote access using Tailscale
    
    ✅ Flask Dashboard

    





##Virtualization & Networking

To create a safe, self-contained lab environment:

    Built VMs with KVM/QEMU

    Created a custom NAT network (virbr1) to:

        Isolate lab traffic from my home Wi-Fi

        Enable tighter control and safer experimentation

    Verified:

        Subnet connectivity and outbound internet access

        Log forwarding over TCP (port 514)


## Firewalls (UFW)
Each VM uses Ubuntu UFW to enforce strict rules:

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
OpenSSH (v6)               ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443 (v6)                   ALLOW       Anywhere (v6)             
514/tcp (v6)               ALLOW       Anywhere (v6)             


## SSH Hardening:
In /etc/ssh/sshd_config 
    
    PasswordAuthentication no

SSH keys are used to authenticate from trusted machines (e.g., my MacBook).

Created a new user account, then set up key-based auth:

    ssh-keygen
    ssh-copy-id user@server


## Centerallized Logging (rsyslog)

Client Configuration (/etc/rsyslog.conf):

Forward all logs to the server over TCP:

    *.* @@<ipadddress>:514
    
Server Configuration:

Enable TCP reception in /etc/rsyslog.conf:

    module(load="imtcp")
    input(type="imtcp" port="514")


![image](https://github.com/user-attachments/assets/2f109d91-bec0-4c3d-980d-243046154261)



## Custom Filters:

In /etc/rsyslog.d/ I created configuration files to:

    Route logs by priority or origin

    Suppress noisy messages

    Store logs in organized directories

    # All logs (including remote) still go to syslog
    *.* /var/log/syslog

Example files:

    10-remote.conf
    20-ufw.conf
    21-cloudinit.conf
    30-sudo-usage.conf
    40-ssh-fail-filter.conf
    50-default.conf

references:

https://www.redhat.com/en/blog/log-aggregation-rsyslog


To make each sure logs of high priority were being addressed, I created configuration rules to direct those logs to a certain location. 

Within /etc/rsyslog.d the following directopries where written 


These are included in the project files.

## Remote Connectivity with Tailscale

Since my private subnet is isolated, port forwarding was not feasible.

I installed Tailscale VPN, which:

    Bridges network gaps

    Provides each device a stable 100.x.x.x address

    Enables secure, authenticated connections from anywhere
    


## Automation with cloud-init

To streamline VM provisioning, I used cloud-init to:

    Configure hostname and users

    Install base packages

    Set up logging

This file can be found in this repo

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


##Log Rotation

Basic log roatation file for safety

/etc/logrotate.d/filtered-logs

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


## Connecting rsyslog to PostgreSQL

I used this as a reference/guide https://www.rsyslog.com/doc/tutorials/database.html

Setup:

    PostgreSQL database serverevents

    Virtual Python environment for my Flask UI

    Ingestion scripts to store and visualize logs

## User Interface

Goal: Create a clean, informative dashboard:

    Built with Flask

    Styled with plain CSS and graphs.css

    Displays logs and visual metrics

Demo Video:



https://github.com/user-attachments/assets/c7c99122-dcb0-4d3d-96da-679493789802


 ##Planned Improvements

    Add an alerting system for critical logs

    Test security using tools like Medusa

    Improve the UI styling

    Tag logs by location/source

    Add authentication to the dashboard

    Containerize the application with Docker
