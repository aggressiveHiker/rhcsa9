# Solutions to Ordered Task List
This file is still in progress...

This shows all of the steps I took for completing each task while preparing for the exam.

1. Break into server2 and set the password as `password`. Set the target as multi-user and make sure it boots into that automatically. Reboot to confirm.
```
Solution:
### While booting into server2, when the GRUB menu screen appears, press the "down" arrow
### to select the rescue kernel, and then press `e` to enter edit mode.

### Append to the end of the line beginning with `linux`
systemd.unit=multi-user.target rd.break enforcing=0

### Ctrl-x to boot into rescue mode
### Enter
mount -o rw,remount /sysroot
chroot /sysroot
passwd root
### password twice
### Ctrl-d twice (just hold ctrl and tap d twice) to fully boot

### Login
restorecon /etc/shadow
systemctl set-default multi-user
systemctl reboot
```

2. Configure the network interfaces and hostnames on both servers.
```
Solution:
### On server1:
nmcli con show

### Output shows `enp0s8` as unconfigured
nmcli con mod enp0s8 ipv4.method manual ipv4.addresses "192.168.55.71/24" ipv4.gateway "192.168.55.1" ipv4.dns "8.8.8.8" ipv6.method manual ipv6.addresses "2002:fe60:def0::55/64"
nmcli con down enp0s8
nmcli con up enp0s8
nmcli general hostname rhcsa9-server1

### Check to make sure everything is good
nmcli con show enp0s8

### On server2:
nmcli con show

### Output shows `enp0s8` as unconfigured
nmcli con mod enp0s8 ipv4.method manual ipv4.addresses "192.168.55.72/24" ipv4.gateway "192.168.55.1" ipv4.dns "8.8.8.8" ipv6.method manual ipv6.addresses "2002:fe60:def0::56/64"
nmcli con down enp0s8
nmcli con up enp0s8
nmcli general hostname rhcsa9-server1

### Check to make sure everything is good
nmcli con show enp0s8
```

3. Ensure network services start at boot.
```
Solution:
systemctl status NetworkManager

### Check to see that it's enabled and running. If not, then run
systemctl enable --now NetworkManager
```

4. Enable ssh access for root on both servers.
```
Solution:
vi /etc/ssh/sshd_config

### Change the following line (should be line 40):
PermitRootLogin yes

systemctl restart sshd
```

5. Enable key-based ssh authentication for root on both servers.
```
Solution:
### On server1:
ssh-keygen
ssh-copy-id root@192.168.55.72
scp /root/.ssh/* root@192.168.55.72:/root/.ssh

### On server2:
ssh-copy-id root@192.168.55.71
```

6. Configure the repos on server1.
```
Solution:
### On server1:
vi /etc/yum.repos.d/local.repo

### Add the below contents to the file:
[BaseOS]
name=BaseOS
enabled=1
baseurl=http://192.168.55.47/repo/BaseOS/
gpgcheck=0

[AppStream]
name=AppStream
enabled=1
baseurl=http://192.168.55.47/repo/AppStream/
gpgcheck=0

[F37]
name=Fedora 37
enabled=0
baseurl=https://download-ib01.fedoraproject.org/pub/fedora/linux/releases/37/Everything/x86_64/os/
gpgcheck=0
```

7. Secure copy the repo file to server2.
```
Solution:
### On server1:
scp /etc/yum.repos.d/local.repo root@192.168.55.72:/etc/yum.repos.d/
```

8. Configure autofs to automatically mount individual users' home directories from `/export/home` on 192.168.55.47 to `/mnt/autofs_home/<user_name>`.
```
Solution:
### On both servers:
dnf install -y autofs
systemctl enable --now autofs
mkdir /mnt/autofs_home

### Add the following line to /etc/auto.master:
/mnt/autofs_home /etc/auto.home

### Create and add the following to /etc/auto.home:
* 192.168.55.47:/export/home/&

systemctl restart autofs
```

9. Configure both servers to create files with 660 permissions by default.
```
Solution:
### On both servers:
vi /etc/login.defs

### Change the following line (should be line 117):
UMASK 007
```

10. Set password policies to require a minimum of 8 characters and a maximum age of 60 days.
```
Solution:
### On both servers:
vi /etc/login.defs

### Change the following line (should be line 131)
PASS_MAX_DAYS   60

vi /etc/security/pwquality.conf

### Uncomment or change the following line (should be line 11):
minlen = 8
```

11. Write shell scripts on server1 that create users and groups according to the following parameters. Ensure all users except cindy use autofs for their profiles:
```
manny:1010:dba_admin,dba_managers,dba_staff
moe:1011:dba_admin,dba_staff
jack:1012:dba_intern,dba_staff
marcia:1013:it_staff,it_managers
jan:1014:dba_admin,dba_staff
cindy:1015:dba_intern,dba_staff
```

12. Secure copy the shell scripts to server2 and perform the same functions.

13. Set the password on all of the newly created users to `dbapass`.

14. Create sudo command alias for `MESSAGES` with the command `/bin/tail -f /var/log/messages`

15. Enable superuser privileges according to the following:
```
dba_managers: everything
dba_admin: SOFTWARE, SERVICES, PROCESSES
dba_intern: MESSAGES
```

16. Switch to the various users using `su` and test their privileges.

17. On server1 create a tar w/gzip archive of /etc called etc_archive.tar.gz in the /archives directory.

18. On server1 create a star w/bzip2 archive of /usr/share/doc called doc_archive.star.bz2 in the /archives directory.

19. On server1 create a folder called /links, and under links create a file called file01. Create a soft link called file02 pointing to file01, and a hard link called file03 pointing to file01. Check your work.

20. Find all setuid files on server1 and save the list to /root/suid.txt.

21. Find all files larger than 3MB in the /etc directory on server1 and copy them to /largefiles.

22. On both servers persistently mount `/export/dba_files` from the server 192.168.55.47 under `/mnt/dba_files`. Ensure manny is the user owner and dba_staff is the group owner. Ensure the groupID is applied to newly created files. Ensure users can only delete files they have created. Ensure only members of the dba_staff group can access the directory.

23. On both servers persistently mount `/export/it_files` from the server 192.168.55.47 under `/mnt/it_files`. Ensure marcia is the user owner and it_staff is the group owner. Ensure the groupID is applied to newly created files. Ensure users can only delete files they have created. Ensure only members of the it_staff group can access the directory.

24. Create a job using `at` to write "This task was easy!" to /coolfiles/at_job.txt in 10 minutes.

25. Create a job using `cron` to write "Wow! I'm going to pass this test!" every Tuesday at 3pm to /var/log/messages.

26. Write a script named awesome.sh in the root directory on server1.
    - a) If “me” is given as an argument, then the script should output “Yes, I’m awesome.”
    - b) If “them” is given as an argument, then the script should output “Okay, they are awesome.”
    - c) If the argument is empty or anything else is given, the script should output “Usage ./awesome.sh me|them”

27. Fix the web server on server1 and make sure all files are accessible. Do not make any changes to the web server configuration files. Ensure it's accessible from server2 and the client browser.

28. Put SELinux on server2 in permissive mode.

29. On server1, modify the bootloader with the following parameters:
```
Increase the timeout using GRUB_TIMEOUT=10
Add the following line: GRUB_TIMEOUT_STYLE=hidden
Add quiet to the end of the GRUB_CMDLINE_LINUX line
```

30. Configure NTP synchronization on both servers. Point them to us.pool.ntp.org.

31. Configure persistent journaling on both servers.

32. On server2, create a new 2GiB volume group on /dev/sdb named "platforms_vg".

33. Under the "platforms_vg" volume group, create a 500MiB logical volume name "platforms_lv" and format it as ext4.

34. Mount it persistently under /mnt/platforms_lv.

35. Extend the "platforms_lv" volume and partition by 500MiB.

36. On server2, create a 500MiB swap partition on /dev/sdb and mount it persistently.

37. On server2, using the remaining space on /dev/sdb, create a volume group with the name networks_vg.

38. Under the "networks_vg" volume group, create a logical volume with the name networks_lv. Ensure it uses 8 MiB extents. Configure the volume to use 75 extents. Format it with the vfat file system and ensure it mounts persistently on /mnt/networks_lv.

39. On server2, create a 5TB thin-provisioned volume on /dev/sdc called "thin_vol" backed by a pool called "thin_pool" on a 5GB volume group called "thin_vg". Format it as xfs and mount it persistently under /mnt/thin_vol.

40. On server1, set a merged tuned profile using the the powersave and virtual-guest profiles.

41. On server1, start one stress-ng process with the niceness value of 19. Adjust the niceness value of the stress process to 10. Kill the stress process.

42. On server1, as the user `cindy`, create a container image from http://192.168.55.47/containers/Containerfile with the tag `cindy_web`.

43. From the newly created image, deploy a container as a service with the container name `cindy_web`. The web config files should map to ~/web_files, and the local port of 8000 should be mapped to the container's port 80. Create a default page that says "Welcome to Cindy's Web Server!". The service should be enabled and the website should be accessible.