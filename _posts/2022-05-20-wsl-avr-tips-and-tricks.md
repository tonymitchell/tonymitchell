---
title: WSL AVR tips and tricks
date: 2022-05-20 11:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, tips, permissions, udev]
---

## #1. Fixing device permissions

Since systemd is not running under WSL2 some of the UDEV rules won't run that set the groups, etc.  As a result the devices under /dev/ will all be owned by root still.

```console
$ ls -al /dev/ttyACM*
crw-rw---- 1 root root 166, 0 May 29 23:14 /dev/ttyACM0
```

As a temporary fix...
Run `sudo system udev restart` BEFORE plugging in the device (or attaching to WSL).  This will ensure that UDEV can make the necessary adjustments. (change the group owner to `plugdev`)

After the fix.
```console
$ ls -al /dev/ttyACM*
crw-rw---- 1 root plugdev 166, 0 May 29 23:14 /dev/ttyACM0
```


Also make sure your user is a member of `plugdev`.  Run `groups` to check

### Making it automatic

Add this to ~/.bashrc
```bash
[ -z "$(pgrep systemd-udevd)" ] && sudo service udev restart &> /dev/null
```

added in sudo visudo to allow .bashrc (or well anyone) to actually do it.).
```
%sudo ALL=NOPASSWD: /usr/sbin/service udev restart 
```

<!--
https://mightyohm.com/blog/2010/03/run-avrdude-without-root-privs-in-ubuntu/
https://hackaday.com/2009/09/18/how-to-write-udev-rules/
https://enotty.pipebreaker.pl/2012/05/23/linux-automatic-user-acl-management/
https://webcache.googleusercontent.com/search?q=cache:Y4cgazdd5i8J:https://blog.luben.se/2021/12/18/wsl-serials.html+&cd=4&hl=en&ct=clnk&gl=ca
-->


<!--

https://unix.stackexchange.com/questions/39370/how-to-reload-udev-rules-without-reboot

Can we do `udevadm trigger --attr-match=subsystem=net` type call instead?

-->

## #2. Force udev to fix permissions without unplugging the device

This seems to be enough get the ACLs set...

If AFTER device already attached... re-trigger event
```bash
sudo service udev restart
sudo udevadm trigger -v --name-match=ttyACM0
```

If BEFORE device attached... this is sufficient
```bash
sudo service udev restart
```

Alternatively you could cycle the detach/attach cycle manually via usbip
```
usbipd.exe wsl detach --hardware-id=2341:0001
usbipd.exe wsl attach --hardware-id=2341:0001
```

## #3. Smart device attachment

Attach if not already attached
```console
[ -z "$(usbipd.exe wsl list | grep 2341:0001.*Attached)" ] && usbipd.exe wsl attach --hardware-id=2341:0001
```

OR grep can do the testing for us...  slightly different semantics... if we find the device and it is not attached...  The other test would try to attach even if the device wasn't listed.
```console
usbipd.exe wsl list | grep -q "2341:0001.*Not attached" && \
echo "Attaching" && \
usbipd.exe wsl attach --hardware-id=2341:0001
```

Crazy one-liner command check for device existence first, and then attach if not already attached.
```console
$ (usbipd.exe wsl list | grep -q " 2341:0001 " || (echo "Device not found"; false)) && (usbipd.exe wsl list | grep -q "2341:0001.*Not attached" && echo "Device found, but not attached. Attaching..." && usbipd.exe wsl attach --hardware-id=2341:0001 || echo "Device attached")
```

> **Broken down...**
>
> Check for device existence
> ```bash
> (usbipd.exe wsl list | grep -q " 2341:0001 " || (echo "Device not found"; false)) && \
> ```
> Check if already attached
> ```bash
> (usbipd.exe wsl list | grep -q "2341:0001.*Not attached" && \
> ```
> If not attached, attach
> ```bash
> echo "Device found, but not attached. Attaching..." && \
> usbipd.exe wsl attach --hardware-id=2341:0001 \
> ```
> Otherwise just report the device attached
> ```bash
> || echo "Device attached")
> ```


## #4. WSL Make enhancements

We can build the smart device attachment into a Makefile target and call it from your flash target automatically.  Now you don't have to worry about usbip at all.

```makefile
#
# WSL features
#
DEVICE_HWID = 2341:0001

# Attach the device via usb/ip if not already attached
usbip_attach:
	@[ -z "$(shell usbipd.exe wsl list | grep $(DEVICE_HWID).*Attached)" ] \
	&& echo "Device not attached. Attaching now..." \
	&& usbipd.exe wsl attach --hardware-id=$(DEVICE_HWID) \
	&& sleep 2 \
	|| true

# Flash via windows-based avrdude.exe
flash_win: AVRDUDE = /mnt/c/DEV/AVR/avrdude/avrdude.exe
flash_win: PROGRAMMER_ARGS = -P COM4
flash_win: flash
```