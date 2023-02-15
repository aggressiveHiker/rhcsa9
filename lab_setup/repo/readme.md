# Repo Server Setup
## Server Specs
- CPU: 2
- Memory: 1GB
- Storage: 32GB
- NIC: 2
  - 1 NAT for internet access
  - 1 Local Host for machine and other server access
- OS: RHEL 9.0 Minimal Install
- IP Address: 192.168.55.47/24

## Repo Setup
Follow dvd_based_repo_creation.md to setup the repos.

Repos will be hosted under:
```
http://192.168.55.47/repo/BaseOS/
http://192.168.55.47/repo/AppStream/
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
http://192.168.55.47/containers/Containerfile
```
Server Location:
```
/var/www/html/containers/Containerfile
```
## Resetting the Server
To reset the server between exam sessions, you should repermission the NFS shares that have been tweaked during the exam tasks, and cleanup the home directories:
```
chown -R root:root /export/{dba_files,it_files}
chmod -R -st,g-w,o+rx /export/{dba_files,it_files}
rm -f /export/home/{cindy,jack,jan,manny,marcia,moe}/.bash_history
rm -fr /export/home/{cindy,jack,jan,manny,marcia,moe}/.cache
rm -fr /export/home/{cindy,jack,jan,manny,marcia,moe}/.config
rm -fr /export/home/{cindy,jack,jan,manny,marcia,moe}/.local
rm -fr /export/home/{cindy,jack,jan,manny,marcia,moe}/.lesshst
```