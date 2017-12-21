# RDM's Linux Build System

This is a simplified version of RDM's system build script. It represents
the core concepts we utilize to build our firmware image.

We begin with using the `debootstrap` utility to build a minimal root
filesystem based on the version of Ubuntu we wish to target. We install
the minimal required to bootstrap that directory to the point where we
can `chroot` to it and continue the package installation and customization
process.

In our full build system we also have processes and support for:
- Specifying a custom kernel version.
- Caching of the required `.deb` files used in the installation process.
  This allows us to build the baseline without any external dependencies.
  We check these packages into our repository, trading repository size
  bloat for the benefit of repeatable builds with zero external dependencies.
  We achieve this by running a custom written proxy interface between the apt
  tool and the remote repositories. This is *not* included in this example
  script but could be added if other teams would like to see how this is
  accomplished
- Local custom repositories of third party `.deb` files such as Adobe Flash.
  This is achieved through the same local deb repository system mentioned
  above. We emulate a `debian repository` for these directories of packages
- Our build process does not end at the creation of the `chroot` directory
  but continues to build a squashfs rootfs image. We then patch the initrd
  scripts to load the squashfs module during initrd and then mount that
  disk image instead of a partition. We overlay this read only squashfs
  root filesystem with a writable root ramdisk of a limited size. This is
  done via a union filsystem driver we load into the initrd. This allows
  for small overlay changes to the rootfs such as changing small
  configuration files. We allow for persistent storage by symbolicly linking
  directories within the overlay-squashfs-root-filesystem to a mounted
  persistent disk partition. Finally we pivot-root to the union filesystem
  and continue the initrd/boot process.
- Our build script emits two main assets we use for deployment, one is a
  gpg signed tar file containing the initrd, boot image (squashfs) and
  kernel. This is in a format we call a 'firmware upgrade' and our devices
  know how to process these files to update our OS partition. Our OS partition
  only contains a few files, a grub configuration, a UEFI boot file, the
  initrd, our boot image (squashfs rootfs), and a kernel. Our build process
  also emits an ISO that, when booted on a device via a thumb drive or CD
  will erase the storage of a device, apply a partitioning scheme, layout
  the necessary files on our OS and data partitions, and apply a valid GRUB
  and UEFI configuration.

We would be happy to discuss those processes as well. However, the script
below, and the similar script for `arm` should describe the core aspect
of our "build from first principals" approach for managing our operating
system needs.

Gavin Stark (gstark@realdigitalmedia.com)

## Running the build

`sudo ./bootstrap-arm`

or

`sudo ./bootstrap-intel`

## Flashing a Jetson TK1 device

- Connect the TK1 to your VM/machine via USB
- Power on the TK1 while keeping the `reset` button pressed (use a pen/paperclip)
- Ensure the `nvidia` device is present (check `lsusb`)
- `sudo ./flash`
- When complete the TK1 may reboot or it may require a power-off/power-on cycle.
