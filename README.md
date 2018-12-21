Note:  This is a work in progress, and does not require rebuilding or compiling. Because of that, it should support *any* processor type including ARM.

If you would rather build OpenWrt, please see the github project https://github.com/mikma/lxd-openwrt (x86 support only)

---------------------------------------------

These scripts create LXD images for OpenWRT.
The resulting image has a couple of problems:
- In order to complete booting, you need to run a script after starting the container.
- interactive ssh to the container does not work.  But non-interactive ssh does.

I hope someone who knows OpenWRT better than I do can fix this.

Naturally, you can run this script inside an LXD container.  I use a privileged Ubuntu 18.04 container, so I can run things like mknod (it is not needed yet).


The openwrt container should be privileged:

	lxc launch {openwrt-image-alias} -c security.privileged=true {name}

The resulting container boots partially.  It becomes usable after running the script /root/init.sh.  You can exec init.sh using lxd exec, either directly, or through an interactive shell:

	lxc exec {container} /root/init.sh

or:

	lxc exec {container} ash
	sh init.sh


init.sh uses mknod to create a few missing devices in /dev.  The container should be privileged in order to be able to run mknod.

I haven't been able to make init.sh run automatically from inside the container.

To stop the container, run "halt" in it.  It does not seem to stop from the LXD tools.

If you try to run "halt" or "reboot" before completing the boot process, it won't work.  In order to get rid of such an unusable container, delete its rootfs from the host (/var/lib/lxd/containers/{container}/rootfs).  You will then be able to delete the container after a host reboot.

The resulting container seems functional.
You can access the Luci Web Interface.

SSH access is limited, as a terminal isn't established, but it is possible to ssh into the container.

```
ssh root@<container> "/bin/sh" -i
BusyBox v1.28.3 () built-in shell (ash)

/root # /bin/sh: can't access tty; job control turned off

/root # 
```

Because of the lack of a `tty` vi does not work very well.

But you can use scp, rsync, and run non-interactive commands with ssh.

The script has been tested with LXD 3.0.2 and OpenWrt 18.06 on a Raspberry Pi running 4.15.0-1029-raspi2 #31-Ubuntu

Thanks to **melato** for pointing me on the right path.


