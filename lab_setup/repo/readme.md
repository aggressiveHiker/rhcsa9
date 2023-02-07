# Repo Server Setup
## Server Specs
- CPU: 2
- Memory: 1GB
- Storage: 32GB
- NIC: 2
  - 1 NAT for internet access
  - 1 Local Host for machine and other server access
- OS: RHEL 9.0 Minimal Install

## Repos
Host the repos under:
```
http://<ip_address>/repo/BaseOS/
http://<ip_address>/repo/AppStream/
```

## Containerfile
Host the Containerfile under:
```
http://<ip_address>/containers/Containerfile
```