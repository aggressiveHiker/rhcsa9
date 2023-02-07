# Repo Server Setup
## Server Specs
- CPU: 2
- Memory: 1GB
- Storage: 32GB
- NIC: 2
  - 1 NAT for internet access
  - 1 Local Host for machine and other server access
- OS: RHEL 9.0 Minimal Install

## Repo Setup
Follow dvd_based_repo_creation.md to setup the repos.

Repos will be hosted under:
```
http://<ip_address>/repo/BaseOS/
http://<ip_address>/repo/AppStream/
```
Server Location:
```
/var/www/html/repo/BaseOS
/var/www/html/repo/AppStream
```

## NFS Setup
Follow nfs_setup.md to setup the NFS server. This will host both regular NFS and autofs for the user profiles.

## Containerfile
Host the Containerfile under:
```
http://<ip_address>/containers/Containerfile
```
Server Location:
```
/var/www/html/containers/Containerfile
```
## Resetting the Server
To reset the server between exam sessions, you should repermission the NFS shares that have been tweaked during the exam tasks:
```
chown -R root:root /export/{dba_files,it_files}
chmod -R -st,g-w,o+rx /export/{dba_files,it_files}
```