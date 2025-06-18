# VM Networking Failure: Troubleshooting and Resolution Summary

## What Happened

A minimal Ubuntu server VM stopped obtaining an IP address. As a result:

- SSH failed (no IP, no route).
- Internet connectivity was lost.
- DHCP client failed silently.
- DNS resolution broke (`resolv.conf` was misconfigured or empty).
- The VM was not connected to an active virtual bridge on the host.

The server was using `systemd-networkd`, not `Netplan` or `NetworkManager`, so traditional tools and configs were unavailable.

## Symptoms

- `ip a` showed `enp1s0` was up but had no IP address.
- `journalctl -u systemd-networkd` showed repeated DHCP lease lost/timeouts.
- SSH gave `No route to host` errors.
- DNS resolution did not work even after networking was partially restored.

## Host-Level Issues

- The host had multiple libvirt networks (`default`, `nat0`, `yellowsubmarine`), and the VM’s NIC was attached to an inactive or misconfigured one.
- Firewall rules and a VPN interfered with DNS and DHCP.
- The libvirt `default` network was inactive.

## Troubleshooting Steps

### Network Validation

```bash
ip a
journalctl -u systemd-networkd

## Libvirt Network Checks (on host)

virsh net-list --all
virsh net-info default
virsh net-info nat0

Default was active and the VM was misconfigured to NAT0

virsh net-start default
virsh net-autostart default

##Restoring DNS Resolution

Manually reconfigured /etc/resolv.conf:

sudo rm /etc/resolv.conf
echo -e "nameserver 8.8.8.8\nnameserver 1.1.1.1" | sudo tee /etc/resolv.conf
sudo systemctl restart systemd-resolved

This allowed package installation and name resolution.
File Transfers During Limited Access

Once an IP was temporarily obtained:

    Used scp and ssh (port 22) for file transfers and access.

scp localfile user@192.168.122.X:/home/user/

##Root Cause Summary

    The VM could not get a DHCP lease due to host-side firewall/VPN interference or network misconfiguration.

    The virtual bridge (virbr2 on nat0) was up but may not have had functional routing.

    The minimal install lacked DHCP clients (dhclient) and troubleshooting tools.

    systemd-networkd was not failing gracefully, leading to no fallback.

    DNS was not functional due to a stale or missing resolv.conf.

##Final Steps Taken

    Reactivated the virtual bridge/network from host.

    Reassigned VM to correct NAT network.

    Set nameservers manually.

    Verified systemd-networkd logs and interface IPs.

    Ensured persistent NAT bridge autostarts.
