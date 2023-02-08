# Notes for RHCSA Exam
## Commands to remember
### General "Getting Around" Commands
- Show the ip address with `ip address show` or `ip a s`
  - With all `ip` commands, the default is `show`, so `ip a` is the same as `ip address show`, and `ip r` is the same as `ip route show`.
  - You can also add colors with the `-c` option for easier readability, such as `ip -c a` or `ip -c r`
- Know the difference between `sudo` and `su`
    - `sudo` stands for "Super User DO" and `su` stands for "Substitute User"
    - `sudo` elevates to root and runs a single command with the current user's password
    - `su` changes to a different user and requires the target user's password
    - `su bob` changes to bob, requires bob's password, but keeps the environment as the source user
    - `su - bob` changes to bob, requires bob's password, and changes the environment to bob's
    - `su -` changes to root because no user is specified, requires the root password, and changes the environment to root's
    - `sudo -i` changes to root with the current user's password
    - `sudo su -` changes to root with the current user's password. Same as `sudo -i`? No, they are not. `sudo -i` temporarily elevates you in interactive mode, but `sudo su -` actually logs you in as root and sets a new login timestamp. If sudo logging is enabled, `sudo -i` will log all commands entered while in interactive mode, but `sudo su -` will only log the initial command of `sudo su -`. All subsequent commands are not logged, becuase now you are logged in as root, rather than running individual commands without having to re-enter your password.
- Generate an ssh key with `ssh-keygen`
- Copy ssh key to other server with `ssh-copy-id user@server`
- Run individual command over ssh with `ssh user@server command`, i.e. to show the version of a remote server, run `ssh user@server cat /etc/redhat-release`
- Copy to or from remote server if you know the path using scp. `scp user@server:/home/user/*.gz /home/user`
- Copy to or from remote server if you don't know the path using sftp. Basic sftp commands are `get`, `put`, and `delete` for individual files, and `mget`, `mput`, and `mdelete` for multiple files. Otherwise, navigation is like in bash. To disconnect, run `close`.
```
sftp user@server
help
ls
ls -la
cd stuff
ls
cd dumps
mget *.gz
### This copies all .gz files to the local current directory
close
```
### TAR and STAR
- `star` is not in any of the default RHEL 9 repos. Add the Fedora 37 repo to get it.
- https://download-ib01.fedoraproject.org/pub/fedora/linux/releases/37/Everything/x86_64/os/
- You can leave the Fedora repo disabled and just point to it directly for the package using the repo id. I.e. `dnf install -y star --repo F37`.

### Documentation Commands
- man
- man -k
- info
- /usr/share/doc

### What is the difference between grep, egrep, and fgrep?
- grep (global regular expression print) is the basic module, but regular expressions require escaping of metacharacters - use it for a basic word search with no special characters
- egrep is extended grep and uses normal regular expressions - use it for a regular expressions search
- fgrep is fixed grep and matches literal strings - use it for a fixed string search that includes special characters

### Managing processes
Use `top`, `ps`, `nice`, `renice`, and `chrt` to manage priority and scheduling. Use `pgrep` and `pkill` to find and kill by name.
- `top` shows the top processes. Press `s` to change the delay, which is 3 seconds by default.
  - You can change a processor priority inside top with renice, by pressing `r`. Lowest priority is 19, highest priority is -20. Remember lower is better.
- `ps` shows running processes... `ps -ef` shows everything... use it with grep
- `nice` launches a command with a specific niceness value.
- `renice` changes the niceness value of an existing process.
- `chrt` changes the real-time attributes of a process
  - `chrt -f -p 45 17350` sets the policy to first-in-first-out, the priority to 45, and applies it to pid 17350
  - Check the current policy on a pid with `chrt -p PID`
  - Default policy will always be set to SCHED_OTHER
- `pgrep` will list pids by name. `pgrep -al` will show you everything...
- `pkill` will kill processes by name

### Logging
- /var/log/messages
- `journalctl`
- `mkdir /var/log/journal` to set up persistent journal logging
- `journalctl --flush` to flush the files from `/run/log/journal` to `/var/log/journal`

### Tuning Profiles
- tuned-adm is the tuned manager
- apply profiles... use --help
- configure dynamic tuning in /etc/tuned/tuned-main.conf

### Repo Management
- find config values in `man dnf.conf`
  - If you can't remember `man dnf.conf`, first to `man -k dnf`, and it will show you all of the listings matching dnf. Go into `man dnf.conf` and then do `G` to go to the end. Page up a couple times and you'll see all of the repo options to drop into .repo files.
- auto-add a repo with `dnf config-manager --add-repo URL`
- After the auto-add, you can go and modify the .repo file created

### Application Streams
- `dnf module list` to show available modules
- `dnf module list php` to narrow it down
- `dnf module install php:8.1/devel` to specify the version and profile
- `dnf module list --installed` to show installed application streams (modules)
- `dnf module list --installed php` to narrow it down
- `dnf module reset php` to reset the profile / disable it
- `dnf module install php:7.5/minimal` to change version and profile after disabling the previous one

### Storage Management
General info
- Use `fdisk` and `gdisk`
- Use `mkfs` and extensions for adding filesystems on partitions `man mkfs` shows `man mkfs.ext4`, etc.
- Don't forget to use the partition number when creating file systems. If you use just the disk name, you'll fuck it up.
- Use `swapon` to manage existing swap
- Use `swapoff` to unmount a swap partition or file
- Use `mkswap` to create new swap

Persistent Mounts
- /etc/fstab
- `lsblk` will show you your current mount tree
- `blkid` will show you the detailed info, including UUID and PARTUUID
- Add the UUID to /etc/fstab to make it persistent. After making it persistent, run `umount <mount_point>` followed by `mount -a` and then check the mounts to make sure they came up.
- `swapon -a` will mount all swap partitions in the fstab

Summary
- `mount` and `umount`
- `swapon` and `swapoff`
- `lsblk` and `blkid`
- `fdisk` and `gdisk`
- `mkfs` and associated sub-commands
- /etc/fstab
- `wipefs -a` is the equivalent of `diskpart clean`
- Extent is like a cluster size and needs to be specified at the volume group level:
  - `vgcreate -s 8MiB myfiles_vg /dev/sdc`
  - You can check it with `vgdisplay` - It's listed under PE Size
  - The default size is 4 MiB

Logical Volume Management
- `pvs` - List physical volumes
- `pvcreate` - Create physical volume
- `vgs` - List volume groups
- `vgcreate` - Create volume group
- `lvs` - List logical volumes
- `lvcreate` - Create logical volume
  - `lvcreate -l 100%FREE -n database1 db_storage` creates a logical volume called database1 under the volume group db_storage

Extend volume group and logical volume
- `pvcreate` the new physical volume
- `vgextend` the current volume group onto the new physical volume
- `lvextend` the current logical volume into the new free space
  - `lvextend -L +1G db_storage/database1` to extend the database1 volume that resides on the db_storage volume group by 1 GB
- `resize2fs` to extend the file system on the logical volume
  - `resize2fs /dev/mapper/db_storage-database1` extends the logical volume

Cleanup / Removal
- `umount /database1`
- `lvremove db_storage/database1`
- `vgremove -f db_storage`
- `pvremove /dev/sdb /dev/sdc`
- `wipefs -a /dev/sdb /dev/sdc`
- Remove mount point from /etc/fstab

### File Systems

nfs
- `/etc/exports` is where you configure the exports
- `/etc/fstab` is where you configure persistent mounts

autofs
- `/etc/autofs.conf` is the main config file. Works out of the box, unless you need to tweak it.
- `/etc/auto.master` is the master map, which points to individual map files
- Map files are configuration files for individual on-demand mount points (e.g. /etc/auto.home)
- The map file points to the file server
  - `/etc/auto.home` contains `* 192.168.55.71:/export/home/&`
- The master file points to the map file
  - `/etc/auto.master` contains `/mnt/nfs_home /etc/auto.home`

### File Permissions

- `umask` sets the default permissions... it tells you which bits to subtract on newly created files. 0002 says to set 775 on directories and 664 on files. 0022 says to set 755 on directories and 644 on files.
- Default out of the box is 0022. Change it globally by creating a shell script under `/etc/profile.d/`, set it on a per-user basis by adding it to `~/.bashrc`. Set it for newly created users only under `/etc/skel/.bashrc`. You can also set it under `/etc/login.defs`.
- `chmod g+s` is the set-GID bit
- `chmod u+s` is the set-UID bit
- `chmod +t` is the sticky bit
- Find files with specific permissions using `find`, e.g. `find / -perm /u+s -exec ls -al {} \; 2>/dev/null` or `find / -perm 777`

### Stratis
- Stratis mounts inside fstab are the same as other mounts with the following caveats:
  - File system is always `xfs`
  - `defaults` is appended with the stratis value, i.e. `defaults,x-systemd.requires=stratisd.service`
- `stratis pool create appteam /dev/sdb` creates the pool named `appteam` on `/dev/sdb`
- `stratis blockdev` shows the stratis pool devices

How to find the mount options, since it's not in `man stratis`
- Think about what fstab does. It mounts stuff.
- Run `man -k mount` and look at the output.
- `fstab` is going to be controlled by `systemd`. If you scan the `man -k` ouput, you'll see `systemd.mount (5)    - Mount unit configuration`
- Run `man 5 systemd.mount`. Once in there, do a find for fstab by doing `/fstab`
- You can see that the first option under the fstab section is `x-systemd.requires=`, well, there's the first half.
- Do a `systemctl status stratisd` and it gives you the second part: `stratisd.service`

### VDO
- VDO in RHEL 9 is created under `lvcreate`, there is information under `man lvmvdo`
- `dnf install -y vdo`
- File system can be `xfs` or `ext4`
- It is dependent on a volume group, like any other logical volume
- `vgcreate vdo-vg /dev/sdb` 
- `lvcreate --type vdo -n web_storage -l 100%FREE -V 30G vdo-vg`
- Hierarchy is Physical Volume >> Volume Group >> Pool >> VDO Logical Volume
- `lvcreate` will also create the pool if you don't specify it, much like `vgcreate` will create the physical volume
- When creating the filesystem on a VDO volume, add the `-K` option
  - `mkfs.ext4 -K /dev/vdo-vg/web_storage`, otherwise the discarding of blocks in the overprovisioned volume will take extra time
- `vdostats` will show you what's going on

### Password Policy
- Password age is configured in `/etc/login.defs`
- Password quality is configured in `/etc/security/pwquality.conf`
- Easy stuff is under `/etc/login.defs`
  - PASS_MAX_DAYS   90
  - PASS_MIN_DAYS   15
  - PASS_WARN_AGE   7

### SSH Security
- Configure it in `/etc/ssh/sshd_conf`
- Enable root login with the `PermitRootLogin yes` value

### Network Management
- `nmcli con show`
- `nmcli con show <name>`
- `nmcli con mod <name> <properties>`
- `nmcli con up|down|reload <name>`

### Managing Superuser Access
- Run `visudo` to edit the /etc/sudoers file. DO NOT EDIT /etc/sudoers directly
- `Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum, /usr/bin/dnf` says the SOFTWARE alias can run those commands
- `%dba_admin      ALL=SOFTWARE, SERVICES, PROCESSES` says the dba_admin group can run commands from those aliases
- The file is pretty well-commented... Easy enough to figure out what to do

### SELinux
- Install everything under keywords `policycoreutils` and `setroubleshoot`
  - `dnf install -y policycore* setrouble*`
- Make enforcement settings persistent in `/etc/selinux/config`
- If you're troubleshooting httpd, for instance, you can find info at the following:
  - `grep httpd /var/log/messages`
  - `grep httpd /var/log/audit/audit.log`
  - `journalctl -u httpd`
  - `ausearch -c 'httpd' --raw`
  - `ausearch -p 1757` (assuming the broken PID from the other logs is 1757)
- After installing setroubleshoot-server, you'll have the `semanage` command
- SELinux port label commands are the following
  - `semanage port -l` to list the ports
  - `semanage port -l | grep http` to narrow it down
  - `semanage port --add -t http_port_t -p tcp 82` to add tcp 82 to the httpd_port_t label

  - /etc/selinux/targetd/contexts/files - This is where the `semanage fcontext` supposedly sends files

### Time, timezone, NTP, hostname
- `timedatectl`
- `/etc/chrony.conf`
- `hostnamectl`
- `nmcli general hostname <hostname>`

### Containers
- `podman` and `skopeo`
- `podman search redis --filter=is-official`
  - Find that in `man podman-search`
- Rootless config https://www.redhat.com/sysadmin/rootless-podman-user-namespace-modes

- `dnf install -y container-tools`
- `less /etc/containers/registries.conf`
- `podman --help | less`
- `podman info | less`
- `podman login registry.access.redhat.com`
- `podman search registry.access.redhat.com/ubi9 | less`
- `podman pull registry.access.redhat.com/ubi9/ubi:latest`
- `podman images` - List images
- `skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:latest | less`
- `podman run -it registry.access.redhat.com/ubi9/ubi:latest`
- `podman ps --all` - List containers
- `podman rm -a` - Remove all containers
- `podman images`
- `podman rmi registry.access.redhat.com/ubi9/ubi:latest` - Remove specific image

Recap
- Basic commands are `podman info`, `podman login`, `podman search`, `podman pull`, `podman images`, `skopeo inspect`, `podman inspect`, `podman run`, `podman ps`, `podman rm`, `podman rmi`...

- `podman run` creates and starts a container
- `podman stop` stops a running container
- `podman start` starts an existing container

```
### Building a couple random Apache containers

skopeo inspect docker://registry.access.redhat.com/rhscl/httpd-24-rhel7 | less
podman pull registry.access.redhat.com/rhscl/httpd-24-rhel7:latest
podman images

### Create a container in detached mode, and map local 8000 to the container's 8080
podman run -d -p 8000:8080 registry.access.redhat.com/rhscl/httpd-24-rhel7

podman ps
firewall-cmd --add-port=8000/tcp
curl http://127.0.0.1:8000
podman stop -a
podman ps -a

### Create an interactive container with /bin/bash
podman run -it registry.access.redhat.com/rhscl/httpd-24-rhel7 /bin/bash

podman ps -a

### Create a detached container with the name test_container
podman run -d --name test_container registry.access.redhat.com/rhscl/httpd-24-rhel7

podman ps -a

### Execute a command against test_container
podman exec -it test_container cat /etc/redhat-release

podman stop test_container
podman ps -a

### Prep local web config files
mkdir ~/web_data
echo "Test data" > ~/web_data/test.txt
cat ~/web_data/test.txt

### Create and run a container called web_container,
### mapping 8000 to 8080 and mapping ~/web_data
### to /var/www/html
podman run -d --name web_container -p 8000:8080 -v ~/web_data:/var/www/html:Z registry.access.redhat.com/rhscl/httpd-24-rhel7

podman ps -a
curl http://127.0.0.1:8000/test.txt
podman ps -a
podman stop -a
```

### Containers as Services
- `loginctl enable-linger`
- `loginctl disable-linger`
- `loginctl show-user <username>`
- `systemctl --user daemon-reload`
- `systemctl --user start|stop|enable UNIT`
- Unit files are at ~/.config/systemd/user/
- `podman generate systemd`

```
### Create and navigate to the directory that will contain the systemd files
mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user

### Create the container with networking and persistent storage
podman run -d --name web_server -p 8000:8080 -v ~/web_data:/var/www/html:Z registry.access.redhat.com/rhscl/httpd-24-rhel7

### Make sure you're in the right directory
pwd
### ~/.config/systemd/user

### Create the systemd config file
podman generate systemd --name web_server --files --new

### Cleanup the container used for the creation
podman stop web_server
podman rm web_server

### Enable the user for linger
loginctl enable-linger
loginctl show-user <username> | grep -i linger

### Add the daemon file and enable the service
systemctl --user daemon-reload
systemctl --user enable --now container-web_server.service

### Check the status
systemctl --user status container-web_server

### Reboot to check persistence
systemctl reboot
```

### Building Container from Containerfile
https://www.youtube.com/watch?v=piwcpd_hWn0

Notes from this video
- `podman container run hello-world`... same as `podman run hello-world`
- `podman container start <container_name>`... same as `podman start <container_name>`
- `podman start -a <container_name>` start in attached mode... attach our console to the container's
- `podman rm <container_name>`
- `podman search <registry>/<keyword>`
- `podman search <fqdn> --list-tags` to show all available versions
- List of registries
  - "registry.fedoraproject.org"
  - "registry.access.redhat.com"
  - "registry.redhat.io" - Requires login
  - "registry.centos.org"
  - "quay.io"
  - "docker.io"
- `podman image tree <image_name>` shows the layers of the image
- `podman run -it <image_name>` -i is Stdin, -t is Stdout. If you run one without the other... well, have fun...
- `podman run -it <image_name> /bin/bash` override the default command with a bash shell
- `podman container prune` remove non-running containers
- `podman run --name something -dit 8080:80 httpd` -dit runs detached with an interactive terminal in case we want to attach later

His version of running as a service (as root):
- `setsebool -P container_manage_cgroup on`
- `podman run -d <everything else...>`
- `podman generate systemd --new --name www`
- `podman generate systemd --new --name www > /etc/systemd/system/www.service`
- `systemctl daemon-reload`
- `systemctl enable --now www`

Here's what we really wanted... building an image...
- As a regular user...
```
mkdir myweb
cd myweb
vi Containerfile
### Paste following:
FROM registry.access.redhat.com/ubi9/ubi:latest
RUN dnf -y install httpd; dnf clean all; systemctl enable httpd;
RUN echo "Successful Web Server Test" | tee /var/www/html/index.html
RUN mkdir /etc/systemd/system/httpd.service.d/; echo -e '[Service]\nRestart=always' | tee /etc/systemd/system/httpd.service.d/httpd.conf
EXPOSE 80
CMD [ "/sbin/init" ]
###
podman image build -t www-image .
```



### Quick steps to deploy from a container as a service
- Make sure to have a fresh login as the user
```
### As root
loginctl enable-linger <uid>

### As user, and make sure it's a fresh login, not `su -`

### Generate the image
podman build -t my_web http://192.168.55.47/containers/Containerfile

### Create the directories for the container and service files
mkdir ~/web_files
mkdir -p ~/.config/systemd/user

### Make sure to be in the service file directory
cd ~/.config/systemd/user

### Start the container with all the parameters
podman run -d --name tony_web -p 8000:80 -v ~/web_files:/var/www/html:Z localhost/my_web

### Generate the service files
podman generate systemd --name tony_web --files --new

### Remove the running container
podman stop tony_web
podman rm tony_web

### Reload the daemons and enable and start the service
systemctl --user daemon-reload
systemctl --user enable --now container-tony_web

### Add some files to the web server and check it out
echo "This server is awesome!" > index.html
echo "Also cool..." > file01
echo "Yep, we did it..." > file02
```
