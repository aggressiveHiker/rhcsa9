# Objective-Based Task List

This task list is for mapping exam objectives to tasks. It is currently in progress and should be ignored.

## Exam Objectives

### Understand and use essential tools
- Access a shell prompt and issue commands with correct syntax
- Use input-output redirection (>, >>, |, 2>, etc.)
- Use grep and regular expressions to analyze text
- Access remote systems using SSH
- Log in and switch users in multiuser targets
- Archive, compress, unpack, and uncompress files using tar, star, gzip, and bzip2
- Create and edit text files
- Create, delete, copy, and move files and directories
- Create hard and soft links
- List, set, and change standard ugo/rwx permissions
- Locate, read, and use system documentation including man, info, and files in /usr/share/doc

### Create simple shell scripts
- Conditionally execute code (use of: if, test, [], etc.)
- Use Looping constructs (for, etc.) to process file, command line input
- Process script inputs ($1, $2, etc.)
- Processing output of shell commands within a script

### Operate running systems
- Boot, reboot, and shut down a system normally

```
EXAM OBJECTIVE - Boot systems into different targets manually
EXAM OBJECTIVE - Interrupt the boot process in order to gain access to a system
TASK --- Break into server2, reset the password, and change the target to multi-user
```
- Identify CPU/memory intensive processes and kill processes
- Adjust process scheduling
- Manage tuning profiles
- Locate and interpret system log files and journals
- Preserve system journals
- Start, stop, and check the status of network services
- Securely transfer files between systems

### Configure local storage
- List, create, delete partitions on MBR and GPT disks
- Create and remove physical volumes
- Assign physical volumes to volume groups
- Create and delete logical volumes
- Configure systems to mount file systems at boot by universally unique ID (UUID) or label
- Add new partitions and logical volumes, and swap to a system non-destructively

### Create and configure file systems
- Create, mount, unmount, and use vfat, ext4, and xfs file systems
```
EXAM OBJECTIVE - Mount and unmount network file systems using NFS
TASK --- Configure NFS on server2
  1. Create home directories in /export/home. Ensure all user's directories exist and are
     permissioned properly and that no data is lost.
  2. Create web data and sales data directories in /export/web_data and /export/sales_data
  3. Persistently mount web data and sales data on server1 under /mnt/nfs_web_data and /mnt/nfs_sales_data
```

```
EXAM OBJECTIVE - Configure autofs
TASK --- Configure autofs on server1 for home directories
  1. Configure autofs on server1 to point to nfs on server2 for home directories using the
     local mount point of /mnt/autofs_home
  2. Ensure users can access their home directories upon login
```
- Extend existing logical volumes

```
EXAM OBJECTIVE - Create and configure set-GID directories for collaboration
TASK --- Configure set-GID on shared directories on server2
```
- Diagnose and correct file permission problems

### Deploy, configure, and maintain systems
- Schedule tasks using at and cron
- Start and stop services and configure services to start automatically at boot
- Configure systems to boot into a specific target automatically
- Configure time service clients

```
EXAM OBJECTIVE - Install and update software packages from Red Hat Network, a remote repository, or from the local file system
TASK --- Configure repositories on both systems and test for functionality
```

```
EXAM OBJECTIVE - Modify the system bootloader
TASK --- 
On server1, make the following changes:
 - Increase the timeout using GRUB_TIMEOUT=10
 - Add the following line: GRUB_TIMEOUT_STYLE=hidden
 - Add quiet to the end of the GRUB_CMDLINE_LINUX line

Validate the changes from the boot screen
```

### Manage basic networking
```
EXAM OBJECTIVE - Configure IPv4 and IPv6 addresses
EXAM OBJECTIVE - Configure hostname resolution
TASK --- Configure IP and DNS settings on both servers according to the system_info doc
```

```
EXAM OBJECTIVE - Configure network services to start automatically at boot
TASK --- Confirm network services start at boot and enable if not configured properly

# systemctl status NetworkManager
# systemctl enable --now NetworkManager
```
- Restrict network access using firewall-cmd/firewall

### Manage users and groups
```
EXAM OBJECTIVE - Configure superuser access
TASK --- Create command alias for MESSAGES with the command `/bin/tail -f /var/log/messages`
TASK --- Enable superuser privileges according to the following:

dba_managers: everything
dba_admin: SOFTWARE, SERVICES, PROCESSES
dba_intern: MESSAGES
```

```
EXAM OBJECTIVE - Create, delete, and modify local user accounts
EXAM OBJECTIVE - Create, delete, and modify local groups and group memberships
TASK --- Create users and groups on both servers according to the following criteria:

manny:1010:dba_admin,dba_managers,dba_staff
moe:1011:dba_admin,dba_staff
jack:1012:dba_intern,dba_staff
marcia:1013:it_staff,it_managers
jan:1014:dba_admin,dba_staff
cindy:1015:dba_intern,dba_staff
```

```
EXAM OBJECTIVE - Change passwords and adjust password aging for local user accounts
TASK --- Set the password on the six newly created accounts to 'dbapass'
TASK --- Set the minimum password length to 8 charachters and the maximum age to 60 days
```

### Manage security
- Configure firewall settings using firewall-cmd/firewalld
- Manage default file permissions

```
EXAM OBJECTIVE - Configure key-based authentication for SSH
TASK --- Enable SSH for root and configure bi-directional key-based authentication
```

```
EXAM OBJECTIVE- Set enforcing and permissive modes for SELinux
TASK --- Persistently set permissive mode for SELinux on server1 and enforcing mode for SELinux on server2
```
- List and identify SELinux file and process context
- Restore default file contexts
- Manage SELinux port labels
- Use boolean settings to modify system SELinux settings
- Diagnose and address routine SELinux policy violations

### Manage containers
- Find and retrieve container images from a remote registry
- Inspect container images
- Perform container management using commands such as podman and skopeo
- Build a container from a Containerfile
- Perform basic container management such as running, starting, stopping, and listing running containers
- Run a service inside a container
- Configure a container to start automatically as a systemd service
- Attach persistent storage to a container