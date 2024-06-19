# NFS Setup
## 1. Install, enable and start NFS
```
dnf install -y nfs-utils
systemctl enable --now nfs-server
```

## 2. Setup Directories
```
mkdir -p /export/{it_files,dba_files,home}
mkdir /export/home/{manny,moe,jack,marcia,jan,cindy}
```

## 3. Add the default profile files
```
for user in manny moe jack marcia jan cindy; do cp /etc/skel/.b* /export/home/$user; done
```

## 4. Change ownership on the profiles
```
chown -R 1010:1010 /export/home/manny
chown -R 1011:1011 /export/home/moe
chown -R 1012:1012 /export/home/jack
chown -R 1013:1013 /export/home/marcia
chown -R 1014:1014 /export/home/jan
chown -R 1015:1015 /export/home/cindy
```

## 5. Setup the exports file
`vi /etc/exports`
```
/export/home         192.168.55.0/24(rw,sync,no_root_squash)
/export/it_files     192.168.55.0/24(rw,sync,no_root_squash)
/export/dba_files    192.168.55.0/24(rw,sync,no_root_squash)
```

## 6. Restart NFS
`systemctl restart nfs-server`

## 7. Add firewall rules
`firewall-cmd --permanent --add-service=nfs --add-service=mountd --add-service=rpc-bind`
