UPDATE: see here for updated images - https://github.com/hexdump0815/imagebuilder

update: some initial bootable ubuntu 18.04 and debian 10 sd card images for the odroid u3, u3+ and x2 are now available at https://github.com/hexdump0815/imagebuilder/releases/tag/190924-01 - see https://github.com/hexdump0815/imagebuilder/blob/master/README.md for some more information about them ...

some notes regarding the odroid u3:

- all this was tested on an odroid u3 with xubuntu 18.04, but should in theory also work on an odroid x2
- see readme.exy for instructions to build the kernel from sources and patches (this is not a shell script)
- random ethernet mac on each boot - fixed with extra kernel cmdline option to set it by hand
- with mali loaded the kernel will crash or mali will not load properly sometimes after reboot - looks like gpu is left in a strange state?
  - disconnecting power _and_ hdmi for a moment (a minute or a few) brings it back at some point
  - without maligpu module the problem does not exist
  - even a fsck restart after an unclean shutdown can trigger it
    - due to its reboot after mali was already loaded
    - so some unplugging might be required here too
- maligpu module unload might result in strange things, so better do not unload it :)
- as the maligpu module seems to be a bit unstable in its usage its best to disable (blacklist) it if not needed
- hopefully lima will take over on a few months as a fully useable replacement for this mali blob solution
- if you need gles or opengl this mali blob solution is at least some kind of working solution for now
- use rk3288 armsoc xorg server for mali (https://github.com/paolosabatino/xf86-video-armsoc.git)
- mali opengl also works with mali-fbdev, modesetting xorg server and gl4es and LIBGL_FB=3, but it is slower
- the odroid u3 seems to be very sensitive to stable power supply
  - if you see strange crashes on boot, check if the power supply is good enough
- suspend/wake/hibernate not tested
- to turn off the blinking blue led: echo none > /sys/class/leds/led1\:heart/trigger
- there is a github kernel tree for the odroid x2 & u3 with quite a few more patches applied to it and updated from time to time at https://github.com/tobiasjakobi/linux-odroid-public - for me it does not give any signal on hdmi and i preferred to stay closer to mainline, but most of the mali stuff in here is from that tree
- on sd cards and emmc better create filesystems with journaling disabled (this should extend their life time)
  - mkfs -t ext4 -O ^has_journal -L somelabel /dev/somedevice

using gles or opengl with special rk3288 armsoc xorg server:
- cd / ; tar xzf xorg-armsoc-armv7l-ubuntu.tar.gz (for ubuntu 18.04)
- cd / ; tar xzf gl4es-armv7l-ubuntu.tar.gz (for ubuntu 18.04)
- cd / ; tar xzf opt-mali-exynos4412.tar.gz
- export LD_LIBRARY_PATH=/opt/mali-exynos4412
- start your gles or opengl app
- opengl is provided via included gl4es libGL.so.1 (https://github.com/ptitSeb/gl4es)
- use 01-armsoc.conf in /etc/X11/xorg.conf.d

using opengl with the default modesetting xorg server and gl4es hack (slower):
- cd / ; tar xzf gl4es-armv7l-ubuntu.tar.gz (for ubuntu 18.04)
- cd / ; tar xzf opt-mali-exynos4412-fbdev.tar.gz
- export LD_LIBRARY_PATH=/opt/mali-exynos4412-fbdev
- export LIBGL_FB=3
- start your opengl app
- opengl is provided via included gl4es libGL.so.1 (https://github.com/ptitSeb/gl4es)
- use 01-modesetting.conf in /etc/X11/xorg.conf.d

mainline u-boot:
- can be built following doc/README.odroid from https://gitlab.denx.de/u-boot/u-boot
- a prebuilt u-boot 2019.07 is provided
  - dd if=u-boot.dd of=/dev/somedevice bs=512 seek=1 skip=1 
  - the seek and skip are to avoid overwriting the partition table
  - leave 16M free before the first partition:
  ```
      fdisk -l
      ...
      Device         Boot   Start      End  Sectors  Size Id Type
      /dev/mmcblk0p1 *      32768  1081343  1048576  512M 83 Linux
      ...
  ```
see the release section of this github repo for a precompiled kernel, modules etc.

maybe interesting forum threads:
* https://forum.odroid.com/viewtopic.php?f=55&t=3691&start=400#p267864
* https://forum.odroid.com/viewtopic.php?f=81&t=9342&start=50#p267863


in case the readme.exy instructions are not clear enough, please have a look at
- https://forum.odroid.com/viewtopic.php?f=55&t=3691&p=276318#p276318
- https://github.com/rodolfoap/kernel-odroid-u3/blob/master/README.md

maybe afterwards its more clear how it is ment to work ...
