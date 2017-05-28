### Overview

In this post I document how to boot an ODROID ARM image from a x86 machine running qemu-system-arm from inside a Docker container.

After this experience, I'm leaning towards a solution that uses binfmt_misc on the host and qemu-arm-static to run ARM docker images as opposed to emulating a whole specific system.  Check out [docker-arm-circleci](https://github.com/philipz/docker-arm-circleci)

*UPDATE*: I just put together an updated system based on the binfmt_misc solution.  Check it out here: [ubergarm/qemu](https://github.com/ubergarm/qemu)


### Moving Parts

1. [Docker](http://docker.io)

While not the first nor the last player in the field, these guys are have definitely garnered serious attention. Docker containers provide a way of adding consistency at different stages whether that be dev, build, or possibly even runtime (there are arm compatible docker images out there `docker search armv7`).

1. [QEMU](http://wiki.qemu.org/)

QEMU has been packaged nicely into a Docker image [tianon/qemu](https://hub.docker.com/r/tianon/qemu/), but that image also requires an `apt-get install -y --no-install-recommends qemu-user-static` for the arm binaries.  For convenience just `docker pull ubergarm/qemu` which provides that single additonal package.

1. [ODROID](http://odroid.com)

These guys have hardware that runs Linux.  Give me bare metal, I'll forge yousomething useful!

### Details

*This guide assumes you already have Docker installed and running.*

#### Prepare desired target image from which to start.

Download and uncompress image file.

    # This server image is smaller and sufficient for testing purposes
    $ wget https://odroid.in/ubuntu_14.04lts/ubuntu-14.04lts-server-odroid-xu3-20150725.img.xz
    # apt-get install p7zip-full or just use `unxz`
    $ 7za e ubuntu-14.04lts-server-odroid-xu3-20150725.img.xz

*NOTE*: This project uses a non [ODROID kernel](https://wiki.linaro.org/ZiShenLim/sandbox/UsingQemuVersatileExpress) with the ODROID image.
I've not been able to successfully boot an actual ODROID kernel/initrd with the currently available QEMU machines.  So instead I've found a newer kernel image that works for a similar Cortex-A15 cpu and use it to bootup the ODROID root file system image. Let me know if you find a valid machine/cpu/kernel/initrd combination!

Download and uncompress image file from which to extract kernel and initrd.

    # we only want the kernel and initrd from which to boot QEMU
    wget http://releases.linaro.org/15.06/ubuntu/vexpress/vexpress-vivid_nano_20150620-722.img.gz
    $ gunzip vexpress-vivid_nano_20150620-722.img.gz

Mount the raw image as a loopback device to extract the kernel and rootfs.

    # look at the img files partition table to check it out sector size
    $ sudo fdisk -lu vexpress-vivid_nano_20150620-722.img
    Disk vexpress-vivid_nano_20150620-722.img: 1073 MB, 1073741824 bytes
    255 heads, 63 sectors/track, 130 cylinders, total 2097152 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
                                Device Boot      Start         End      Blocks   Id  System
    vexpress-vivid_nano_20150620-722.img1   *          63      155646       77792    e  W95 FAT16 (LBA)
    vexpress-vivid_nano_20150620-722.img2          155648     2097151      970752   83  Linux

    # prepare mount point
    $ mkdir image
    # mount it and specify an offset of (Start * sector size) to skip partition table
    $ sudo mount -t auto -o loop,offset=$((63*512)) vexpress-vivid_nano_20150620-722.img ./image

*NOTE*: If you want to acces the root fs partition it would look something like this:

    $ sudo mount -t auto -o loop,offset=$((155648*512)) vexpress-vivid_nano_20150620-722.img ./image

Confirm that the desired files are living in the boot partition.

    $ ls -la ./image/
    -rwxr-xr-x 1 root root 4844744 Jun 20 07:16 uImage
    -rwxr-xr-x 1 root root 2093436 Jun 20 07:16 uInitrd
    -rwxr-xr-x 1 root root   12981 Jun 20 07:16 v2p-ca15-tc1.dtb
    # Copy the kernel image, and the ram disk image, and device tree blob out.
    $ cp image/uImage .
    $ cp image/uInitrd .
    $ cp image/v2p-ca15-tc1.dtb .
    # Unmount
    $ sudo unmount image

Manually [unpack](http://stackoverflow.com/questions/22322304/image-vs-zimage-vs-uimage) the kernel and ramdisk images by removing u-boot wrapper (if necessary).

    $ sudo apt-get install u-boot-tools
    # confirm the kernel and initrd have u-boot headers
    $ mkimage -l uImage
    Image Name:   Linux
    Created:      Sat Jun 20 02:16:22 2015
    Image Type:   ARM Linux Kernel Image (uncompressed)
    Data Size:    4844680 Bytes = 4731.13 kB = 4.62 MB
    Load Address: 60008000
    Entry Point:  60008000
    $ mkimage -l uInitrd
    Image Name:   uInitrd 3.10.82
    Created:      Sat Jul 25 02:41:05 2015
    Image Type:   ARM Linux RAMDisk Image (uncompressed)
    Data Size:    2116820 Bytes = 2067.21 kB = 2.02 MB
    Load Address: 00000000
    Entry Point:  00000000
    # strip out the 64-byte header
    $ dd if=uImage of=kernel bs=64 skip=1
    $ dd if=uInitrd of=initrd bs=64 skip=1
    # inspect resulting files
    $ file kernel
    kernel: Linux kernel ARM boot executable zImage (little-endian)
    $ file initrd
    initrd: gzip compressed data, from Unix, last modified: Sat Jun 20 02:14:35 2015

#### Attempt to boot using qemu from inside a Docker container.

Pull the a Docker image with qemu and qemu-user-static installed.

    $ docker pull ubergarm/qemu

Run a container with required files volume mounted into it.

    # can also pass in --device /dev/kvm if you need it
    $ docker run --rm -it -v `pwd`:/home/qemu ubergarm/qemu /bin/bash

Now from inside the container give it a go:

    # get into the volume mounted directory with the files
    $ cd /home/qemu
    # boot with the kernel/ramdisk to run on the target image root file system
    qemu-system-arm \
        -M vexpress-a15 \
        -cpu cortex-a15 \
        -m 512 \
        -net nic \
        -net user,hostname=qemu,hostfwd=tcp::22-:22,hostfwd=udp::22-:22 \
        -nographic \
        -kernel kernel \
        -initrd initrd \
        -dtb v2p-ca15-tc1.dtb \
        -sd ubuntu-14.04lts-server-odroid-xu3-20150725.img \
        -append "root=/dev/mmcblk0p2 rw mem=512M raid=noautodetect console=ttyAMA0 rootwait devtmpfs.mount=0"

#### Success!?!

    Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 4.1.0-1-linaro-vexpress armv7l)

    * Documentation:  https://help.ubuntu.com/

    System information disabled due to load higher than 1.0

    0 packages can be updated.
    0 updates are security updates.

    * Stopping System V initialisation compatibility                        [ OK ]
    * Starting System V runlevel compatibility                              [ OK ]
    * Starting deferred execution scheduler                                 [ OK ]
    * Starting regular background program processing daemon                 [ OK ]
    * Starting ACPI daemon                                                  [ OK ]
    * Starting save kernel messages                                         [ OK ]
    * Stopping save kernel messages                                         [ OK ]
    * Starting automatic crash report generation                            [ OK ]
    * Stopping System V runlevel compatibility                              [ OK ]
    * Starting OpenSSH server                                               [ OK ]
    root@odroid-server:~# uname -a
    Linux odroid-server 4.1.0-1-linaro-vexpress #1ubuntu1~ci+150619154554-Ubuntu SMP Fri Jun 19 16:13:03 UTC 201 armv7l armv7l armv7l GNU/Linux
    root@odroid-server:~# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/mmcblk0p2  3.1G  743M  2.2G  26% /
    udev            248M  4.0K  248M   1% /dev
    tmpfs            50M  232K   50M   1% /run
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    none            5.0M     0  5.0M   0% /run/lock
    none            249M     0  249M   0% /run/shm
    none            100M     0  100M   0% /run/user
    /dev/mmcblk0p1  100M  5.2M   95M   6% /media/boot
    root@odroid-server:/lib# ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
        valid_lft forever preferred_lft forever

### SSH into QEMU

The configuration above sets up the QEMU environment to accept and forward port 22 (SSH).  You can expose that port however you would like from the Docker container or just access it directly e.g.:

    # from outside the Docker container
    $ docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CID>
    172.17.0.1
    $ ssh root@172.17.0.1
    root@172.17.0.1's password:
    Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 4.1.0-1-linaro-vexpress armv7l)
    * Documentation:  https://help.ubuntu.com/
    System information disabled due to load higher than 1.0
    0 packages can be updated.
    0 updates are security updates.
    Last login: Mon Nov 23 20:58:20 2015
    root@odroid-server:~#


### Conclusion

Hopefully here you have an ODROID image booting up from your PC.

I wish the ODROID kernel would boot so everything would match up better, but I've heard gossip is a group interested in supporting an ODROID machine in QEMU... :)

Still need to figure out how to best keep the image clean (read only/cleanly unmount and halt/automate fsck)...  Calling `halt` then doing a `docker stop` and waiting might work for now.

Finally, you may need to edit the /etc/resolv.conf in your ARM image to lookup domain names.
