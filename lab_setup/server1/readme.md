# Server 1 Setup
## Server Specs
- CPU: 2
- Memory: 2GB
- Storage: 32GB
- NIC: 2
  - 1 NAT for internet access
  - 1 Local Host for machine and other server access
- OS: RHEL 9.0 Minimal Install

## Software
- Install httpd and break it to perform firewall and SELinux tasks
```
vi /etc/httpd/conf/httpd.conf and change Listen 80 to Listen 82

echo "RHCSA is Awesome" > /var/www/html/file1

cal > /var/www/html/file2

echo "This file is totally messed up" > /var/www/html/file3

chcon -t default_t /var/www/html/file3
```