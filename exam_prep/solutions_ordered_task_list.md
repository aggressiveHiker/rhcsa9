# Solutions to Ordered Task List
This guide shows all of the steps I took for completing each task while preparing for the exam.

**1.** Break into server2 and set the password as `password`. Set the target as multi-user and make sure it boots into that automatically. Reboot to confirm.

#### **Solution to Task 1**
```
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

**2.** Configure the network interfaces and hostnames on both servers.

#### **Solution to Task 2**
```
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

**3.** Ensure network services start at boot.

#### **Solution to Task 3**
```
systemctl status NetworkManager

### Check to see that it's enabled and running. If not, then run
systemctl enable --now NetworkManager
```

**4.** Enable ssh access for root on both servers.

#### **Solution to Task 4**
```
vi /etc/ssh/sshd_config

### Change the following line (should be line 40):
PermitRootLogin yes

systemctl restart sshd
```

**5.** Enable key-based ssh authentication for root on both servers.

#### **Solution to Task 5**
```
### On server1:
ssh-keygen
ssh-copy-id root@192.168.55.72
scp /root/.ssh/* root@192.168.55.72:/root/.ssh

### On server2:
ssh-copy-id root@192.168.55.71
```

**6.** Configure the repos on server1.

#### **Solution to Task 6**
```
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

**7.** Secure copy the repo file to server2.

#### **Solution to Task 7**
```
### On server1:
scp /etc/yum.repos.d/local.repo root@192.168.55.72:/etc/yum.repos.d/
```

**8.** Configure autofs to automatically mount individual users' home directories from `/export/home` on 192.168.55.47 to `/mnt/autofs_home/<user_name>`.

#### **Solution to Task 8**
```
### On both servers:
dnf install -y nfs-utils autofs
systemctl enable --now autofs
mkdir /mnt/autofs_home

### Add the following line to /etc/auto.master:
/mnt/autofs_home /etc/auto.home

### Create and add the following to /etc/auto.home:
* 192.168.55.47:/export/home/&

### Set SELinux boolean to allow for NFS home directories ###
setsebool -P use_nfs_home_dirs on

systemctl restart autofs
```

**9.** Configure both servers to create files with 660 permissions by default.

#### **Solution to Task 4**
```
### On both servers:
vi /etc/login.defs

### Change the following line (should be line 117):
UMASK 007
```

**10.** Set password policies to require a minimum of 8 characters and a maximum age of 60 days.

#### **Solution to Task 10**
```
### On both servers:
vi /etc/login.defs

### Change the following line (should be line 131)
PASS_MAX_DAYS   60

vi /etc/security/pwquality.conf

### Uncomment or change the following line (should be line 11):
minlen = 8
```

**11.** Write shell scripts on server1 that create users and groups according to the following parameters. Ensure all users except cindy use autofs for their profiles:
```
manny:1010:dba_admin,dba_managers,dba_staff
moe:1011:dba_admin,dba_staff
jack:1012:dba_intern,dba_staff
marcia:1013:it_staff,it_managers
jan:1014:dba_admin,dba_staff
cindy:1015:dba_intern,dba_staff
```

#### **Solution to Task 11**
On server1, create a text file called "grouplist.txt" that contains the following:
```
dba_admin:5010
dba_managers:5011
dba_staff:5012
dba_intern:5013
it_staff:5014
it_managers:5015
```
Then create a shell script called "creategroup.sh" that contains the following:\
Note: "IFS" is internal field separator, and will separate the group names from the group IDs.
```
#!/bin/bash

while IFS=":" read group gid ; do
echo "Creating group $group..."
groupadd -g $gid $group
done < grouplist.txt
```
Set the permissions on the file so it's executable, then run it:
```
chmod +x creategroup.sh
./creategroup.sh
```
Now check that the groups were created:
```
egrep "it|dba" /etc/group
```
Now that the groups are created, create a text file called "userlist.txt" that contains the following:\
Note: Remember that "cindy" is being created differently, so needs to be omitted.
```
manny:1010:dba_admin,dba_managers,dba_staff
moe:1011:dba_admin,dba_staff
jack:1012:dba_intern,dba_staff
marcia:1013:it_staff,it_managers
jan:1014:dba_admin,dba_staff
```
Create a shell script called "createuser.sh" that contains the following:\
Note: Here we still use the IFS, and we specify the home path with `-b`, the auxilary groups with `-G` and not to create a home directory with `-M`.
```
#!/bin/bash

while IFS=":" read user uid group ; do
echo "Creating user $user..."
useradd -b /mnt/autofs_home -G $group -u $uid -M $user
done < userlist.txt
```
Set the permissions on the file so it's executable, then run it:
```
chmod +x createuser.sh
./createuser.sh
```
Check to make sure the users were created and added to the appropriate groups:
```
tail /etc/passwd
egrep "it|dba" /etc/group
```
Finally, create "cindy" as a one-off user with a local profile:
```
useradd -u 1015 -G dba_intern,dba_staff cindy
```

**12.** Secure copy the shell scripts to server2 and perform the same functions.

#### **Solution to Task 12**
```
scp create* root@192.168.55.72:/root/
scp *.txt root@192.168.55.72:/root/

### Now on server2:
./creategroup.sh
./createuser.sh
useradd -u 1015 -G dba_intern,dba_staff cindy
```

**13.** Set the password on all of the newly created users to `dbapass`.

#### **Solution to Task 13**
```
### On both servers:

for user in manny moe jack marcia jan cindy; do echo "dbapass" | passwd --stdin $user; done
```

**14.** Create sudo command alias for `MESSAGES` with the command `/bin/tail -f /var/log/messages`

#### **Solution to Task 14**
```
### On both servers:

visudo

### While in visudo, add the following lines:

## Messages
Cmnd_Alias MESSAGES = /bin/tail -f /var/log/messages
```

**15.** Enable superuser privileges according to the following:
```
dba_managers: everything
dba_admin: SOFTWARE, SERVICES, PROCESSES
dba_intern: MESSAGES
```

#### **Solution to Task 15**
```
### On both servers:

visudo

### While in visudo, uncomment the following lines:

Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start....
Cmnd_Alias PROCESSES = /bin/nice, /bin/kill, /usr/bin/kill, /usr/bin/killall

### Then add the following to the mapping section, after %wheel%:

%dba_managers  ALL=(ALL)       ALL
%dba_admin     ALL = SOFTWARE, SERVICES, PROCESSES
%dba_intern    ALL = MESSAGES
```

**16.** Switch to the various users using `su` and test their privileges.

#### **Solution to Task 16**
```
### manny is a dba_manager, so he should have all rights:
su - manny
sudo -i
### If he can elevate, it works

### moe is a dba_admin, but not a dba_manager:
su - moe
sudo -i (this should fail)
sudo yum install tree (don't confirm, but this should work)

### jack is a dba_intern, but not a dba_admin or dba_manager:
su - jack
sudo -i (this should fail)
sudo yum install tree (this should fail)
sudo tail -f /var/log/messages

### marcia is not a member of any elevated groups:
su - marcia
sudo -i (this should fail)
sudo yum install tree (this should fail)
sudo tail -f /var/log/messages (this should fail)
```

**17.** On server1 create a tar w/gzip archive of /etc called etc_archive.tar.gz in the /archives directory.

#### **Solution to Task 17**
```
dnf install -y tar gzip
tar -czvf /archives/etc_archive.tar.gz /etc
```

**18.** On server1 create a star w/bzip2 archive of /usr/share/doc called doc_archive.star.bz2 in the /archives directory.

#### **Solution to Task 18**
```
dnf install -y star --repo F37
dnf install -y bzip2
star -c -v -j file=/archives/doc_archive.star.bz2 /usr/share/doc
```

**19.** On server1 create a folder called /links, and under links create a file called file01. Create a soft link called file02 pointing to file01, and a hard link called file03 pointing to file01. Check your work.

#### **Solution to Task 19**
```
mkdir /links
touch /links/file01
ln -s /links/file01 /links/file02
ln /links/file01 /links/file03
ls -lai /links
```

**20.** Find all setuid files on server1 and save the list to /root/suid.txt.

#### **Solution to Task 20**
```
find / -type f -perm -u+s > /root/suid.txt
cat suid.txt
```

**21.** Find all files larger than 3MB in the /etc directory on server1 and copy them to /largefiles.

#### **Solution to Task 21**
```
mkdir /largefiles
find /etc -type f -size +3M -exec cp {} /largefiles \; 2>/dev/null
ls -al /largefiles/
```

**22.** On both servers persistently mount `/export/dba_files` from the server 192.168.55.47 under `/mnt/dba_files`. Ensure manny is the user owner and dba_staff is the group owner. Ensure the groupID is applied to newly created files. Ensure users can only delete files they have created. Ensure only members of the dba_staff group can access the directory.

#### **Solution to Task 22**
```
mkdir /mnt/dba_files
vi /etc/fstab

### Add the following line to /etc/fstab:
192.168.55.47:/export/dba_files  /mnt/dba_files  nfs  defaults  0 0

### Write and quit /etc/fstab, then check the mount:
mount -a

### Set the permissions:
chown manny:dba_staff /mnt/dba_files
chmod 770 /mnt/dba_files
chmod g+s,+t /mnt/dba_files
```

**23.** On both servers persistently mount `/export/it_files` from the server 192.168.55.47 under `/mnt/it_files`. Ensure marcia is the user owner and it_staff is the group owner. Ensure the groupID is applied to newly created files. Ensure users can only delete files they have created. Ensure only members of the it_staff group can access the directory.

#### **Solution to Task 23**
```
mkdir /mnt/it_files
vi /etc/fstab

### Add the following line to /etc/fstab:
192.168.55.47:/export/it_files  /mnt/it_files  nfs  defaults  0 0

### Write and quit /etc/fstab, then check the mount:
mount -a

### Set the permissions:
chown marcia:it_staff /mnt/it_files
chmod 770 /mnt/it_files
chmod g+s,+t /mnt/it_files
```

**24.** Create a job using `at` to write "This task was easy!" to /coolfiles/at_job.txt in 10 minutes.

#### **Solution to Task 24**
```
dnf install -y at
systemctl enable --now atd

at now + 10 minutes
mkdir /coolfiles
echo "This task was easy!" > /coolfiles/at_job.txt
#Ctrl-d to exit
```

**25.** Create a job using `cron` to write "Wow! I'm going to pass this test!" every Tuesday at 3pm to /var/log/messages.

#### **Solution to Task 25**
```
vi /etc/crontab

### Add the following line to /etc/crontab
0 15 * * 2 root echo "Wow! I'm going to pass this test!" >> /var/log/messages
```

**26.** Write a script named awesome.sh in the root directory on server1.
- a) If “me” is given as an argument, then the script should output “Yes, I’m awesome.”
- b) If “them” is given as an argument, then the script should output “Okay, they are awesome.”
- c) If the argument is empty or anything else is given, the script should output “Usage ./awesome.sh me|them”

#### **Solution to Task 26**
```
vi /awesome.sh
```
Add the following to the file:
```
#!/bin/bash

if [ "$1" = "me" ] ; then
    echo "Yes, I'm awesome."

elif [ "$1" = "them" ] ; then
    echo "Okay, they are awesome."

else
    echo "Usage ./awesome.sh me|them"

fi
```
Change the permissions and test it:
```
chmod +x /awesome.sh
/awesome.sh me
/awesome.sh them
/awesome.sh everyone
/awesome.sh
```

**27.** Fix the web server on server1 and make sure all files are accessible. Do not make any changes to the web server configuration files. Ensure it's accessible from server2 and the client browser.

#### **Solution to Task 27**
Ok, this is a long one. First, we need to check the web server and see what's going on:
```
systemctl status httpd
```
We see something similar to the following:
```
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: failed (Result: exit-code) since Wed 2023-02-08 14:03:47 CET; 7s ago
       Docs: man:httpd.service(8)
    Process: 2935 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 2935 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 37ms

Feb 08 14:03:47 rhcsa9-server1 systemd[1]: Starting The Apache HTTP Server...
Feb 08 14:03:47 rhcsa9-server1 httpd[2935]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:2>
Feb 08 14:03:47 rhcsa9-server1 httpd[2935]: (13)Permission denied: AH00072: make_sock: could not bind to address [::]:85
Feb 08 14:03:47 rhcsa9-server1 httpd[2935]: (13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:85
Feb 08 14:03:47 rhcsa9-server1 httpd[2935]: no listening sockets available, shutting down
Feb 08 14:03:47 rhcsa9-server1 httpd[2935]: AH00015: Unable to open logs
Feb 08 14:03:47 rhcsa9-server1 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Feb 08 14:03:47 rhcsa9-server1 systemd[1]: httpd.service: Failed with result 'exit-code'.
Feb 08 14:03:47 rhcsa9-server1 systemd[1]: Failed to start The Apache HTTP Server.
```
Ok, we can see here that the it had `(13)Permission denied`, and could not bind to port 85. So, we know the webserver is on port 85, and it's having a permissions issue. Port 85 is a reserved port, and SELinux isn't going to let us bind to that port. Let's get some tools installed to troubleshoot:
```
dnf install -y policycore* setrouble*
```
Now that we have our tools installed, let's try to start it again and see what our logs tell us:
```
systemctl start httpd
grep httpd /var/log/messages
```
Oh man, look at all that gold...
```
Feb  8 14:03:47 rhcsa9-server1 httpd[2935]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:27ff:fe6b:9f4a%enp0s3. Set the 'ServerName' directive globally to suppress this message
Feb  8 14:03:47 rhcsa9-server1 httpd[2935]: (13)Permission denied: AH00072: make_sock: could not bind to address [::]:85
Feb  8 14:03:47 rhcsa9-server1 httpd[2935]: (13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:85
Feb  8 14:03:47 rhcsa9-server1 httpd[2935]: no listening sockets available, shutting down
Feb  8 14:03:47 rhcsa9-server1 httpd[2935]: AH00015: Unable to open logs
Feb  8 14:03:47 rhcsa9-server1 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Feb  8 14:03:47 rhcsa9-server1 systemd[1]: httpd.service: Failed with result 'exit-code'.
Feb  8 14:03:50 rhcsa9-server1 setroubleshoot[2936]: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 85. For complete SELinux messages run: sealert -l d28fbbf2-b7c7-4a6d-a677-7b498dc14c8c
Feb  8 14:03:50 rhcsa9-server1 setroubleshoot[2936]: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 85.#012#012*****  Plugin bind_ports (99.5 confidence) suggests   ************************#012#012If you want to allow /usr/sbin/httpd to bind to network port 85#012Then you need to modify the port type.#012Do#012# semanage port -a -t PORT_TYPE -p tcp 85#012    where PORT_TYPE is one of the following: http_cache_port_t, http_port_t, jboss_management_port_t, jboss_messaging_port_t, ntop_port_t, puppet_port_t.#012#012*****  Plugin catchall (1.49 confidence) suggests   **************************#012#012If you believe that httpd should be allowed name_bind access on the port 85 tcp_socket by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -X 300 -i my-httpd.pp#012
Feb  8 14:03:52 rhcsa9-server1 setroubleshoot[2936]: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 85. For complete SELinux messages run: sealert -l d28fbbf2-b7c7-4a6d-a677-7b498dc14c8c
Feb  8 14:03:52 rhcsa9-server1 setroubleshoot[2936]: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 85.#012#012*****  Plugin bind_ports (99.5 confidence) suggests   ************************#012#012If you want to allow /usr/sbin/httpd to bind to network port 85#012Then you need to modify the port type.#012Do#012# semanage port -a -t PORT_TYPE -p tcp 85#012    where PORT_TYPE is one of the following: http_cache_port_t, http_port_t, jboss_management_port_t, jboss_messaging_port_t, ntop_port_t, puppet_port_t.#012#012*****  Plugin catchall (1.49 confidence) suggests   **************************#012#012If you believe that httpd should be allowed name_bind access on the port 85 tcp_socket by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -X 300 -i my-httpd.pp#012
```
If we read through all of that, we can see a couple of things going on there. First, it tells us that SELinux is preventing httpd from binding to the port, and then tells us if we want even more detail, we can run `sealert -l d28fbbf2-b7c7-4a6d-a677-7b498dc14c8c`. We don't need to do that, since there's enough info to get going with. Looking further down, it gives us a couple of suggestions on how to fix it. First, we can change the port type by running `semanage port -a -t PORT_TYPE -p tcp 85` and specifying the port type, *OR...* we can create a custom policy by running `ausearch -c 'httpd' --raw | audit2allow -M my-httpd` and then follow it with `semodule -X 300 -i my-httpd.pp`. We're going to choose the latter:
```
ausearch -c 'httpd' --raw | audit2allow -M my-httpd
semodule -X 300 -i my-httpd.pp
```
Now try starting the service again:
```
systemctl start httpd
```
Alright! It works!

Next, we're going to check the files locally:
```
ls -la /var/www/html

total 12
drwxr-xr-x. 3 root root  57 Feb  7 10:46 .
drwxr-xr-x. 4 root root  33 Feb  1 08:40 ..
-rw-r--r--. 1 root root  18 Feb  1 08:42 file1
-rw-r--r--. 1 root root 168 Feb  1 08:43 file2
-rw-r--r--. 1 root root  32 Feb  1 08:43 file3
```
Okay, we have three files we need to hit here. Let's give it a shot:
```
curl http://127.0.0.1:85/file1

RHCSA is Awesome!
```
So far, so good.
```
curl http://127.0.0.1:85/file2

    February 2023
Su Mo Tu We Th Fr Sa
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28
```
Still doing alright... Let's check the last one.
```
curl http://127.0.0.1:85/file3

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```
Damn! We were so close...

Ok, let's see if we can figure out what's going on here. Back to the logs...
```
grep httpd /var/log/messages
```
```
Feb  8 14:26:36 rhcsa9-server1 setroubleshoot[3227]: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/file3. For complete SELinux messages run: sealert -l d618c7b8-64e2-42ff-8a6e-03e1cf089356
Feb  8 14:26:36 rhcsa9-server1 setroubleshoot[3227]: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/file3.#012#012*****  Plugin restorecon (94.8 confidence) suggests   ************************#012#012If you want to fix the label. #012/var/www/html/file3 default label should be httpd_sys_content_t.#012Then you can run restorecon. The access attempt may have been stopped due to insufficient permissions to access a parent directory in which case try to change the following command accordingly.#012Do#012# /sbin/restorecon -v /var/www/html/file3#012#012*****  Plugin catchall_labels (5.21 confidence) suggests   *******************#012#012If you want to allow httpd to have getattr access on the file3 file#012Then you need to change the label on /var/www/html/file3#012Do#012# semanage fcontext -a -t FILE_TYPE '/var/www/html/file3'#012where FILE_TYPE is one of the following: NetworkManager_exec_t, NetworkManager_log_t, NetworkManager_tmp_t, ...OUTPUT_OMITTED... keystone_cgi_content_t, keystone_cgi_htac
Feb  8 14:26:37 rhcsa9-server1 setroubleshoot[3227]: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/file3. For complete SELinux messages run: sealert -l d618c7b8-64e2-42ff-8a6e-03e1cf089356
Feb  8 14:26:37 rhcsa9-server1 setroubleshoot[3227]: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/file3.#012#012*****  Plugin restorecon (94.8 confidence) suggests   ************************#012#012If you want to fix the label. #012/var/www/html/file3 default label should be httpd_sys_content_t.#012Then you can run restorecon. The access attempt may have been stopped due to insufficient permissions to access a parent directory in which case try to change the following command accordingly.#012Do#012# /sbin/restorecon -v /var/www/html/file3#012#012*****  Plugin catchall_labels (5.21 confidence) suggests   *******************#012#012If you want to allow httpd to have getattr access on the file3 file#012Then you need to change the label on /var/www/html/file3#012Do#012# semanage fcontext -a -t FILE_TYPE '/var/www/html/file3'#012where FILE_TYPE is one of the following: NetworkManager_exec_t, NetworkManager_log_t, NetworkManager_tmp_t, ...OUTPUT_OMITTED... keystone_cgi_content_t, keystone_cgi_htac
```
Ok, just like last time, it's showing us what's going on here and giving suggestions. SELinux is preventing httpd from getattr access to the file. It suggests we can either `restorecon` the file, or run `semanage fcontext -a -t FILE_TYPE '/var/www/html/file3'` to change the label altogether. Let's take a look at the directory:
```
ls -laZ /var/www/html

drwxr-xr-x. 3 root root system_u:object_r:httpd_sys_content_t:s0      57 Feb  7 10:46 .
drwxr-xr-x. 4 root root system_u:object_r:httpd_sys_content_t:s0      33 Feb  1 08:40 ..
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0  18 Feb  1 08:42 file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 168 Feb  1 08:43 file2
-rw-r--r--. 1 root root unconfined_u:object_r:default_t:s0            32 Feb  1 08:43 file3
```
And we can see that file3 has a different label from the other two. Let's try the `restorecon`:
```
restorecon /var/www/html/file3
```
And check the directory again:
```
ls -laZ /var/www/html

drwxr-xr-x. 3 root root system_u:object_r:httpd_sys_content_t:s0      57 Feb  7 10:46 .
drwxr-xr-x. 4 root root system_u:object_r:httpd_sys_content_t:s0      33 Feb  1 08:40 ..
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0  18 Feb  1 08:42 file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 168 Feb  1 08:43 file2
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0  32 Feb  1 08:43 file3
```
Alright, it's looking better! Now let's try to curl it:
```
curl http://127.0.0.1:85/file3

This file is totally messed up!
```
Success!!! Alright, last thing is to fix the firewall so external clients can reach it:
```
firewall-cmd --add-port=85/tcp --permanent
firewall-cmd --reload
```

**28.** Put SELinux on server2 in permissive mode.

#### **Solution to Task 28**
```
vi /etc/selinux/config

### Change the following line:
SELINUX=permissive
```

**29.** On server1, modify the bootloader with the following parameters:
```
Increase the timeout using GRUB_TIMEOUT=10
Add the following line: GRUB_TIMEOUT_STYLE=hidden
Add quiet to the end of the GRUB_CMDLINE_LINUX line
```

#### **Solution to Task 29**
```
vi /etc/default/grub

### Add or edit the following lines:
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=hidden
GRUB_CMDLINE_LINUX  ### add quiet to the end

### Write and quit the file

grub2-mkconfig -o /boot/grub2/grub.cfg

### Reboot and watch the boot from the console to verify
systemctl reboot
```

**30.** Configure NTP synchronization on both servers. Point them to us.pool.ntp.org.

#### **Solution to Task 30**
```
### On both servers:
vi /etc/chrony.conf

### Edit the following line (should be line 3):
pool us.pool.ntp.org iburst

### Write and quit, then restart the service:
systemctl restart chronyd

### Check the logs to ensure time is being pulled from the new source:
journalctl -u chronyd

### You should see a line similar to the following at the end:
Feb 14 09:00:48 rhcsa9-server1 chronyd[705]: Selected source 73.61.36.59 (us.pool.ntp.org)
```

**31.** Configure persistent journaling on both servers.

#### **Solution to Task 31**
```
### On both servers:
mkdir /var/log/journal
journalctl --flush
ls -la /var/log/journal
```

**32.** On server2, create a new 2GiB volume group on /dev/sdb named "platforms_vg".

#### **Solution to Task 32**
First, create an LVM partition in fdisk:
```
### Enter fdisk
fdisk /dev/sdb

### n for new partition
n

### p for primary
p

### 1 for partition number
1

### Press enter to select the first available sector

### +2G for setting the last sector as 2GiB past the first sector
+2G

### t to change the partition type
t

### l to list all partition types
l

### 8e or lvm to select lvm (lvm is an alias for 8e)
lvm

### w to write and quit
w
```

Then, create the volume group:
```
vgcreate platforms_vg /dev/sdb1
```

**33.** Under the "platforms_vg" volume group, create a 500MiB logical volume name "platforms_lv" and format it as ext4.

#### **Solution to Task 33**
```
lvcreate -L 500M -n platforms_lv platforms_vg
mkfs.ext4 /dev/mapper/platforms_vg-platforms_lv
```

34. Mount it persistently under /mnt/platforms_lv.

#### **Solution to Task 34**
```
mkdir /mnt/platforms_lv

### use lsblk -f to get the UUID
lsblk -f

### Add the UUID to /etc/fstab with the mount point
vi /etc/fstab

UUID=<uuid_from_lsblk>  /mnt/platforms_lv  ext4  defaults  0 0

### Write and quit, then mount all to check that it mounts properly
mount -a
mount
```

**35.** Extend the "platforms_lv" volume and partition by 500MiB.

#### **Solution to Task 35**
```
### Run an lvextend including the -r switch to extend both the volume and filesystem
lvextend -L +500M -r /dev/mapper/platforms_vg-platforms_lv
```

**36.** On server2, create a 500MiB swap partition on /dev/sdb and mount it persistently.

#### **Solution to Task 36**
First, create a swap partition in fdisk:
```
### Enter fdisk
fdisk /dev/sdb

### n for new partition
n

### p for primary
p

### 2 for partition number
2

### Press enter to select the first available sector

### +500M for setting the last sector as 500MiB past the first sector
+500M

### t to change the partition type
t

### l to list all partition types
l

### 82 or swap to select swap (swap is an alias for 82)
swap

### w to write and quit
w
```

Next, create the swap on the new partition:
```
mkswap /dev/sdb2
```

Last, find the UUID and add it to /etc/fstab:
```
### use lsblk -f to get the UUID
lsblk -f

### Add the UUID to /etc/fstab with the mount point
vi /etc/fstab

UUID=<uuid_from_lsblk>  swap  swap  defaults  0 0

### Write and quit, then activate all swap to check that it mounts properly
swapon -a
swapon
```

**37.** On server2, using the remaining space on /dev/sdb, create a volume group with the name networks_vg.

#### **Solution to Task 37**
First, create an LVM partition in fdisk:
```
### Enter fdisk
fdisk /dev/sdb

### n for new partition
n

### p for primary
p

### 3 for partition number
3

### Press enter to select the first available sector

### Press enter to select the last sector

### t to change the partition type
t

### l to list all partition types
l

### 8e or lvm to select lvm (lvm is an alias for 8e)
lvm

### w to write and quit
w
```

Now, we need to create the volume group, but looking ahead to task 38, it says the logical volume needs to be using 8MiB extents, which we need to configure when we create the volume group:
```
vgcreate -s 8M networks_vg /dev/sdb3
```

**38.** Under the "networks_vg" volume group, create a logical volume with the name networks_lv. Ensure it uses 8 MiB extents. Configure the volume to use 75 extents. Format it with the vfat file system and ensure it mounts persistently on /mnt/networks_lv.

#### **Solution to Task 38**
```
### Create the volume
lvcreate -l 75 -n networks_lv networks_vg

### Install dosfstools and create the file system
dnf install -y dosfstools
mkfs.vfat /dev/mapper/networks_vg-networks_lv

### Find the UUID with lsblk
lsblk -f

### Create the directory
mkdir /mnt/networks_lv

### Add the UUID to /etc/fstab
vi /etc/fstab

UUID=<uuid_from_lsblk>  /mnt/networks_lv  vfat  defaults  0 0

### Write and quit, then mount all to check
mount -a
mount
```

**39.** On server2, create a 5TB thin-provisioned volume on /dev/sdc called "thin_vol" backed by a pool called "thin_pool" on a 5GB volume group called "thin_vg". Format it as xfs and mount it persistently under /mnt/thin_vol.

#### **Solution to Task 39**
For this task, we are using VDO within LVM to create the volume.

First, create the partition for the volume group in fdisk:
```
### Enter fdisk
fdisk /dev/sdc

### n for new partition
n

### p for primary
p

### 1 for partition number
1

### Press enter to select the first available sector

### +5G for setting the last sector as 5GiB past the first sector
+5G

### t to change the partition type
t

### l to list all partition types
l

### 8e or lvm to select lvm (lvm is an alias for 8e)
lvm

### w to write and quit
w
```

Then, create the volume group:
```
vgcreate thin_vg /dev/sdc1
```

Install VDO, create the volume, and format it as XFS:
```
dnf install -y vdo
lvcreate --vdo -l 100%FREE -V 5T --name thin_vol thin_vg/thin_pool

### Use the -K flag to prevent the discarding of blocks
mkfs.xfs -K /dev/mapper/thin_vg-thin_vol
```

Last, add the volume to /etc/fstab and mount it
```
### Find the UUID with lsblk
lsblk -f

### Create the directory
mkdir /mnt/thin_vol

### Add the UUID to /etc/fstab
vi /etc/fstab

UUID=<uuid_from_lsblk>  /mnt/thin_vol  xfs  defaults  0 0

### Write and quit, then mount all to check
mount -a
mount
```

**40.** On server1, set a merged tuned profile using the the powersave and virtual-guest profiles.

#### **Solution to Task 40**
```
dnf install -y tuned
systemctl enable --now tuned
tuned-adm profile powersave virtual-guest
```

**41.** On server1, start one stress-ng process with the niceness value of 19. Adjust the niceness value of the stress process to 10. Kill the stress process.

#### **Solution to Task 41**
```
dnf install -y stress-ng
nice -n 19 stress-ng -c 1 &

### Enter top and renice the process
top
r
### Select the pid and enter 10
q

pkill stress-ng
```

**42.** On server1, as the user `cindy`, create a container image from http://192.168.55.47/containers/Containerfile with the tag `web_image`.

#### **Solution to Task 42**
As root, enable linger for cindy:
```
loginctl enable-linger cindy
```
Login separately as cindy. It needs to be a fresh login, do not use `su -`. Then build the image:
```
podman build -t web_image http://192.168.55.47/containers/Containerfile
```

If we run a `podman images` now, we should see the newly created image:
```
podman images

### Output:
REPOSITORY                           TAG         IMAGE ID      CREATED       SIZE
localhost/web_image                  latest      b14526ab4f3a  1 minute ago  243 MB
registry.access.redhat.com/ubi9/ubi  latest      10acc174412e  9 days ago    219 MB
```

**43.** From the newly created image, deploy a container as a service with the container name `cindy_web`. The web config files should map to ~/web_files, and the local port of 8000 should be mapped to the container's port 80. Create a default page that says "Welcome to Cindy's Web Server!". The service should be enabled and the website should be accessible.

#### **Solution to Task 43**
First, login separately as cindy. It needs to be a fresh login, do not use `su -`.

Now, let's go ahead and create the local directory for the web files and the default web page for our web server:
```
mkdir web_files
echo "Welcome to Cindy's Web Server!" > web_files/index.html
```

Next, let's fix the general permissions on the directory and file, because otherwise it will be inaccessible (remember the umask change from task 4):
```
chmod 755 web_files
chmod 644 web_files/index.html
```

Next, we can list the available images by doing `podman images`:
```
podman images

### Output:
REPOSITORY                           TAG         IMAGE ID      CREATED       SIZE
localhost/web_image                  latest      b14526ab4f3a  1 minutes ago 243 MB
registry.access.redhat.com/ubi9/ubi  latest      10acc174412e  9 days ago    219 MB
```

Now, let's go ahead and kick off a container that does everything. We're going to do the `podman run` command to spin it up, the `-d` switch to run it in detached mode, the `-p` switch to map our local port to our internal container port, the `-v` switch to map our local storage to our internal container storage, the `:Z` flag to create a custom SELinux label on the files, the `--name` switch to name the container, and finally specify the image we'll be running:
```
podman run -d -p 8000:80 -v ~/web_files:/var/www/html:Z --name cindy_web localhost/web_image
```

We can now see the running container by doing a `podman ps`:
```
podman ps

### Output:
CONTAINER ID  IMAGE                       COMMAND     CREATED        STATUS            PORTS                 NAMES
fb89491cdf08  localhost/web_image:latest  /sbin/init  1 minutes ago  Up 1 minutes ago  0.0.0.0:8000->80/tcp  cindy_web
```

Ok, let's test and make sure we can reach it:
```
curl http://127.0.0.1:8000

### Output:
Welcome to Cindy's Web Server!
```

Now, let's build a service off the container. First we need to create the directory for the service files, and navigate to it:
```
mkdir -p .config/systemd/user
cd .config/systemd/user
```

Next, we need to build the service files of the running container:
```
podman generate systemd --files --name cindy_web --new

### Check and see the file created:
ls -la

### Output:
drwxrwx---. 3 cindy cindy  69 Feb 15 19:35 .
drwxrwx---. 3 cindy cindy  18 Feb 15 19:34 ..
-rw-r--r--. 1 cindy cindy 792 Feb 15 19:35 container-cindy_web.service
```

Now, before we enable the service, we have to stop and remove the running container:
```
podman stop cindy_web
podman rm cindy_web
```

Enable and start the service with the `--user` switch:
```
systemctl --user enable --now container-cindy_web.service
```

Now if we check again, we'll see the container running:
```
podman ps

### Output:
CONTAINER ID  IMAGE                       COMMAND     CREATED        STATUS            PORTS                 NAMES
5bf2d72dbfa1  localhost/web_image:latest  /sbin/init  4 seconds ago  Up 5 seconds ago  0.0.0.0:8000->80/tcp  cindy_web
```

We can also check the status of the service:
```
systemctl --user status container-cindy_web

### Output:
● container-cindy_web.service - Podman container-cindy_web.service
     Loaded: loaded (/home/cindy/.config/systemd/user/container-cindy_web.service; enabled; vendor preset: disabled)
     Active: active (running) since Fri 2023-02-17 11:02:07 CET; 2min 38s ago
       Docs: man:podman-generate-systemd(1)
    Process: 1728 ExecStartPre=/bin/rm -f /run/user/1015/container-cindy_web.service.ctr-id (code=exited, status=0/SUCCESS)
   Main PID: 1757 (conmon)
      Tasks: 15 (limit: 11108)
     Memory: 25.5M
        CPU: 211ms
     CGroup: /user.slice/user-1015.slice/user@1015.service/app.slice/container-cindy_web.service
             ├─1741 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp --enable-ipv6 -c -e 3 >
             ├─1743 rootlessport
             ├─1749 rootlessport-child
             └─1757 /usr/bin/conmon --api-version 1 -c 5bf2d72dbfa18ef8f250ff36775e2e8c4a6e79dbc418bf007e079fbeab540069 -u 5bf2d72dbf>

Feb 17 11:02:07 rhcsa9-server1 systemd[915]: Starting Podman container-cindy_web.service...
Feb 17 11:02:07 rhcsa9-server1 podman[1729]:
Feb 17 11:02:07 rhcsa9-server1 podman[1729]: 2023-02-17 11:02:07.632856538 +0100 CET m=+0.112055029 container create 5bf2d72dbfa18ef8>
Feb 17 11:02:07 rhcsa9-server1 podman[1729]: 2023-02-17 11:02:07.684918995 +0100 CET m=+0.164117448 container init 5bf2d72dbfa18ef8f2>
Feb 17 11:02:07 rhcsa9-server1 systemd[915]: Started Podman container-cindy_web.service.
Feb 17 11:02:07 rhcsa9-server1 podman[1729]: 2023-02-17 11:02:07.690819129 +0100 CET m=+0.170017582 container start 5bf2d72dbfa18ef8f>
Feb 17 11:02:07 rhcsa9-server1 podman[1729]: 5bf2d72dbfa18ef8f250ff36775e2e8c4a6e79dbc418bf007e079fbeab540069
Feb 17 11:02:07 rhcsa9-server1 podman[1729]: 2023-02-17 11:02:07.596761866 +0100 CET m=+0.075960345 image pull  localhost/web_image
```

Alright! Everything looks good!

The last thing we need to do is to update the firewall (as root):
```
### As root
firewall-cmd --add-port=8000/tcp --permanent
firewall-cmd --reload
```

Congratulations!!! You're ready for your RHCSA 9 exam!
