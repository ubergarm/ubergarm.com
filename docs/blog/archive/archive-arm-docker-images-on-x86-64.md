The goal is to automate building armhf (armv7) compatible Docker images. These images can be used for dev, build, test, and deploy to ensure a consistent user experience.  The most common development, cloud, and build machines currently run x86-64 architecture hardware.  This complicates working on software targeted at embedded arm platforms. With modern kernels that support binfmt misc and the right combination of qemu-XXX-static binary one can run arbitrary format Docker images on arbitrary host hardware.

This solution combines:

* Host kernel support for binfmt misc
* qemu-arm-static
* Docker armhf images

## Configure Host

There are a many articles out there on configuring binfmt / qemu / chroot to run ARM binaries, images, etc.  Here are two common approaches.

### The Clean Way

Assuming a newer Ubuntu or Debian system you can just install the necessary pacakges.

    apt-get update && apt-get install -y --no-install-recommends \
            qemu-user-static \
            binfmt-support
    update-binfmts --enable qemu-arm
    update-binfmts --display qemu-arm

If everything looks good, skip ahead past the next section down to *Test Host Setup*.

### The Dirty Way

If you want to keep it slim or manually configure everything this would be another approach.

Grab a host compatible qemu-arm-static binary and put it in place.

    sudo curl -o /usr/bin/qemu-arm-static http://ubergarm.com/dre/qemu-arm-static
    sudo chmod a+x /usr/bin/qemu-arm-static

Install binfmt module support if not already built into kernel.

    grep BINFMT /usr/src/linux-headers-$(uname -r)/.config || zcat /proc/config.gz | grep BINFMT
    sudo modprobe binfmt_misc
    lsmod | grep BINFMT

Register qemu-arm 

    mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
    echo ':qemu-arm:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/usr/bin/qemu-arm-static:' > /proc/sys/fs/binfmt_misc/register

### Test Host Setup

Confirm that the host is setup correctly to run an arm binary.

    qemu-arm-static -version
    curl -O http://ubergarm.com/dre/hello-world-arm
    chmod a+x hello-world-arm
    file hello-world-arm
    qemu-arm-static hello-world-arm
    ./hello-world-arm

If all that looks good e.g. "Hello world!" both times then try Docker.

    docker pull ioft/armhf-ubuntu:trusty
    docker run --rm -it -v /usr/bin/qemu-arm-static:/usr/bin/qemu-arm-static ioft/armhf-ubuntu:trusty uname -a

You should get something like:

    Linux 27393a16e4f8 3.13.0-68-generic #111-Ubuntu SMP Fri Nov 6 18:17:06 UTC 2015 armv7l armv7l armv7l GNU/Linux

as compared to the host `uname -a`:

    Linux qemu-test 3.13.0-68-generic #111-Ubuntu SMP Fri Nov 6 18:17:06 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

## NOTE

Not all arm images have an x86-64 qemu-arm-static sitting around in /usr/bin already. If it isn't there, just volume mount the file from the host into place in the container as shown above. Until `docker build -v` works, I've gone ahead and created a base image which only adds qemu-arm-static to an otherwise ARM Docker Image to support Dockerfile's and building Docker Images from an x86. 

    $ docker run --rm -it ubergarm/armhf-ubuntu:trusty file /usr/bin/qemu-arm-static

## Conclusion

There you have it!  Run, build, test, and dev ARM Docker Images right on your x86-64 host machine.  Next up: bringing Continuous Integration to the embedded arm world!

## Plug

I tested all of the above commands for just a few cents on a fresh clean [DigitalOcean](https://www.digitalocean.com/?refcode=e1f614cb2907) VPS. Use that referral link to get $10 credit if you want so try it out yourself.  If you keep using it, they'll eventually give me $25 credit. Thanks. Plug complete.

## References
[ioft/armhf-ubuntu](https://hub.docker.com/r/ioft/armhf-ubuntu/)

[ubergarm/armhf-ubuntu](https://hub.docker.com/r/ubergarm/armhf-ubuntu/)
