# Server 2 Setup
## Server Specs
- CPU: 2
- Memory: 2GB
- Storage: 3 Drives
  - 1x 32GB for BaseOS Install
  - 2x 16GB for storage tasks
- NIC: 2
  - 1 NAT for internet access
  - 1 Local Host for machine and other server access
- OS: RHEL 9.0 Minimal Install

## Break the boot
Set the root password to something random, and then break the boot so it doesn't even get to a login prompt:
```
systemctl set-default network.target
systemctl reboot
```