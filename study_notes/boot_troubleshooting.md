## How to manipulate the boot sequence
### General Info
- `systemctl get-default` will show you the default boot target
  - `multi-user.target` is the CLI-only boot environment
  - You can find the complete list of targets at `ls /usr/lib/systemd/system/*.target`
- The official way to reboot is `systemctl reboot`
- Likewise, shutdown with `systemctl poweroff`

### Setting verbose messages
- Remove `rhgb quiet` from /etc/default/grub
- Run `grub2-mkconfig -o /boot/grub2/grub.cfg`

### Breaking into the machine
- While at the grub screen, press `e` to break the sequence. This will take you into "edit" mode.
- At the end of the line starting with `linux`, append `rd.break`
- Hit Ctrl-x to boot into a rescue shell using the edited config

Once in the emergency shell, run the following commands:
```
### Remount /sysroot as read-write
mount -o rw,remount /sysroot

### Change root to /sysroot
chroot /sysroot

### Change the password
passwd root

### Force selinux to autorelabel everything
touch /.autorelabel

### Exit the /sysroot shell
exit

### Remount /sysroot as read-only
mount -o ro,remount /sysroot

### Exit emergency mode
exit
```

After it reboots, press `e` to enter the grub edit screen again. At the end of the line starting with `linux`, append `systemd.unit=multi-user.target` to force it to boot into CLI-only mode.

Once booted in and logged in, permanently change the target to multi-user.target and then check it
```
systemctl set-default multi-user.target
systemctl get-default
```
## OR...
### Doing it all in a single boot
Alternatively, you can do it in a single boot by using the `enforcing=0` switch after `rd.break`. This will let you login a single time without relabeling. To make it persistent, you need to restore the SELinux security context.

At the grub edit screen, append the following to the line beginning with `linux`:
```
systemd.unit=multi-user.target rd.break enforcing=0
```

Once in the emergency shell, run the following commands:
```
### Remount /sysroot as read-write
mount -o rw,remount /sysroot

### Change root to /sysroot
chroot /sysroot

### Change the password
passwd root

### Exit the /sysroot shell
exit

### Remount /sysroot as read-only
mount -o ro,remount /sysroot

### Exit emergency mode
exit
```

Once booted in and logged into the system with the new root password, restore the SELinux security context:
```
### Restore the security context
restorecon /etc/shadow

### Turn SELinux policy back on and check it
setenforce 1
getenforce
```

After fixing the labeling, change and check the default target:
```
systemctl set-default multi-user.target
systemctl get-default
```

Definitely give it a reboot to make sure everything is working properly...

### IMPORTANT NOTE
On RHEL 9.0 (9.0 only... this is fixed in 9.1), you must select the rescue kernel from the GRUB2 menu before hitting `e`, otherwise you will be prompted for a root password to get into rescue mode.
https://bugzilla.redhat.com/show_bug.cgi?id=2057365

### Absolute Fastest Method
```
### On RHEL 9.0, press the down arrow to select the rescue kernel
### Then press `e` to enter edit mode

### Append to linux line...
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