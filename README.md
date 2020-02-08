INSTALLING ARCH LINUX
=====================

Copyright (C) Adrian Ratnapala, 2019.

The [installation guide][archinst] for [Arch Linux][arch] is
https://wiki.archlinux.org/index.php/Installation_guide; that guide
intentionally leaves many choices up to the reader.  This document is about how
to set up Arch the way I, Adrian, like it.

[arch]: https://www.archlinux.org
[archinst]: https://wiki.archlinux.org/index.php/Installation_guide


Making a boot stick
-------------------

### Get the ISO

You can [download][archdl] an ISO using BitTorrent or from a mirror.  I used
[syd.mirror.rackspace.com], but got the sigs from a few places.

    VER=2020.02.01
    curl -o aarnet.sig https://mirror.aarnet.edu.au/pub/archlinux/iso/$VER/archlinux-$VER-x86_64.iso.sig
    curl -o rackspace.sig https://mirror.rackspace.com/archlinux/iso/$VER/archlinux-$VER-x86_64.iso.sig
    curl -o china.sig http://mirrors.cqu.edu.cn/archlinux/iso/$VER/archlinux-$VER-x86_64.iso.sig
    curl -O https://syd.mirror.rackspace.com/archlinux/iso/$VER/archlinux-$VER-x86_64.iso.sig
    sha256sum *.sig

Which yields

    bfa41224e009642e3ceeb2469bebb4a4034ca36c74a9465bd7dc86c47250f6d0  aarnet.sig
    bfa41224e009642e3ceeb2469bebb4a4034ca36c74a9465bd7dc86c47250f6d0  archlinux-2020.02.01-x86_64.iso.sig
    bfa41224e009642e3ceeb2469bebb4a4034ca36c74a9465bd7dc86c47250f6d0  china.sig
    bfa41224e009642e3ceeb2469bebb4a4034ca36c74a9465bd7dc86c47250f6d0  rackspace.sig


They match, good.

    curl -O https://syd.mirror.rackspace.com/archlinux/iso/$VER/archlinux-$VER-x86_64.iso

[archdl]: https://www.archlinux.org/download/
[arch-at-sydrack]: https://syd.mirror.rackspace.com/archlinux/iso/

### Verify that ISO

I don't like or understand GPG enough to know what to do with the `.sig` file, except that since I am
doing this on an existing arch installation, I run.

    pacman-key -v archlinux-$VER-x86_64.iso.sig

This checks the `.sig` against the corresponding file (in the same directory) and says:

<pre>
==> Checking archlinux-2020.02.01-x86_64.iso.sig... (detached)
gpg: Signature made Sat 01 Feb 2020 17:57:48 AEDT
gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
gpg: Note: trustdb not writable
gpg: Good signature from "Pierre Schmitz <pierre@archlinux.de>" [full]
</pre>

#### How do you know the mirror handed you a real `.sig`?

We can go to https://www.archlinux.org/people/developers/ and see that pierre@
is there; and his pgp key [0x9741E8AC links][pierre-key] with a URL containing
the string `0x4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC`.

[pierre-key]: https://sks-keyservers.net/pks/lookup?op=vindex&fingerprint=on&exact=on&search=0x4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC

This is not proof, but it will do for me.

### Flash the ISO to a stick

Easy peasy for nerds, and since this is he 21st century, we even have a
progress indicator.

<pre>
VER=VER
dd status=progress bs=8192000 if=archlinux-$VER-x86_64.iso of=<b>/SOME USB DEVICE/</b>
</pre>

Booting into the live environment
---------------------------------

You run the installer that you just flashed by booting of the stick.  How this
is done depends on your machine.  See my notes for

* [Lenovo Thinkpad X1 Carbon 5th Gen (20HRA04JAU)](thinkpad-x1-5th)

One you the boot menu from the stick, select **Boot Arch Linux**.  This will
get you into a live Arch Linux environment that you can use to install the
system, and also as a rescue disc.


## Make the live environment usable

At this point you will be following
https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation which
is mostly about making the live environment itself usable.  Just do it.

Internet connectivity is the only potential trouble. My two-cents are:

* **Use a wired connection**, the live environment will probably just work
  (using DHCP) as soon as you plug it in.
* One time I needed to **boot without the cable plugged in**.  But the problem
  did not recur. YYMV.

Disks
-----

I want to encrypt my as much as possible, but still boot simply.  So I have
decided for `scooter19` to:

* Have an unencrypted `/boot`, this is the same thing as the [(U)EFI
  parition](uefi).  This parition shipped partition 1, with the volume label
  `SYSTEM`.
* Put the rest of the disk on encrypted [LUKS][luks] volume `cryptlvm`
* Map this 1-1 onto a [LVM][lvm] volume group `cryptvg`, on which I have
  - A 50G volume `ROOT` (`/`, `/var` and `/usr`)
  - A 20G volume called `SWAP`, not used, but reserved for when I do hibernate-to-disk
  - The rest is `HOME`, for `/home`

The main authority for the encryption stuff is
[https://wiki.archlinux.org/index.php/Dm-crypt][luks].
Which has a sub-page for this kind of [LVM-on-LUKS][lvm-on-luks] setup.

[LVM]: https://wiki.archlinux.org/index.php/LVM
[luks]: https://wiki.archlinux.org/index.php/Dm-crypt
[lvm-on-luks]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS

### Partitioning

I did the partitioning with `fdisk`, but dumped the result using

    sfdisk -d /dev/nvme0n1 > MOUNTED_EFI_PARITION/2019-04/nvme0n1-after.dump

Yielding

        label: gpt
        label-id: AF64EACC-EC26-44BA-81FA-A8924D5C663D
        device: /dev/nvme0n1
        unit: sectors
        first-lba: 34
        last-lba: 1000215182

        /dev/nvme0n1p1 : start=        2048, size=      532480, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=F8D454E8-800B-46A0-8353-ABE9169B9FBE, name="EFI system partition", attrs="RequiredPartition GUID:63"
        /dev/nvme0n1p2 : start=      534528, size=   999680655, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=E8C7DE15-C7C2-9D41-934E-F07302F2C283

### Formatting etc.

Because the `/boot` partition is shared outside of Linux, I have not formatted
it.  That made it a nice place to keep logs of what I was doing.  The following
is not exactly a shell script, but I could uncomment whichever line I wanted
and then run it:

    luks+lvm
    --------
    DEV=/dev/nvme0n1p2
    #cryptsetup luksFormat --type luks2 "$DEV" # interactive, asks for passphrase
    #cryptsetup open "$DEV" cryptlvm # interactive, asks for passphrase
    cryptsetup open UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03 cryptlvm
    echo $/dev/mapper
    ls /dev/mapper -l

    LVM_DEV=/dev/mapper/cryptlvm
    #pvcreate "$LVM_DEV"
    #vgcreate cryptvg "$LVM_DEV"

    #lvcreate -L 20G cryptvg -n SWAP
    #lvcreate -L 50G cryptvg -n ROOT
    #lvcreate -l 100%FREE cryptvg -n HOME

    #mkfs.ext4 -v /dev/cryptvg/ROOT -L ROOT
    #mkfs.ext4 -v /dev/cryptvg/HOME -L HOME
    # Do no make swap yet.

Notice the crypt device has two different UUIDs:

* `E8C7DE15-C7C2-9D41-934E-F07302F2C283` is the *GPT partition* UUID
  and is used in device names like `PARTUUID=....` and
  `/dev/disk/by-partuuid/...`

* `f550c62-63b0-49fc-804a-3b4b112bcc03` was inserted by `luksFormat` and Linux
   seems to consider it the "real" UUID -- we can name the device with
   `UUID=...` or `/dev/disk/by-uuid/....

There is a similar distniction between volume labels and GPT parition labels.
The salient difference is that if you `dd` the an image to some other device, you
will retain the UUID and LABEL, but not the PARTUUID and PARTLABEL.


Setting up the installed OS
---------------------------

### Mounts

The convention is that we mount all our filesystems under `/mnt` in a pattern
which mirrors the mount on the final system.  On scooter19 that meant:

    mount /dev/cryptvg/ROOT /mnt -L ROOT
    mkdir /mnt/home
    mount /dev/cryptvg/HOME /mnt/home -L HOME
    mount /dev/by-label
    mkdir /mnt/boot
    mount /dev/disk/by-label/SYSTEM efi # a.k.a /dev/nvme0n1p1

### Installing the packages

Now we will install base packages, downloading them from repositories defined
in `/etc/pacman.d/mirrorlist`.  You can edit that file to use mirrors near
you, see:
https://wiki.archlinux.org/index.php/Installation_guide#Select_the_mirrors

And install with

     pacstrap /mnt base


### Post-package setup

    genfstab -L /mnt >> /mnt/etc/fstab
    arch-chroot /mnt

Sets us up in some approximation of the real system (`-L` to genfstab uses
label-based names, `-U` uses UUIDs).

There are many other straightforward instructions to follow at
https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system

Booting
-------

There are many options, and I have chosen:

* **Use busybox initrd** (the default), as opposed to systemd.

* **Using [EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB)**.  This
  turns the Kernel into standard EUFI executable that can be run from its
  shell, or directly from the boot menu.

* **Booting the kernel directly from the boot menu**.  The alternative is to
  write a [EUFI shell script][nsh] booted from the menu.  I have such a script,
  which was useful for debugging, but it doesn't work from the menu -- perhaps
  because I haven't installed a shell yet.

What's tricky is to get all this to work in the face the encrypted disk.  Even
though the [instructions][dm-crypt-sysconfig] were correct.

[dm-crypt-sysconfig]: https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration

### mkinitcpio

[mkinitcpio][mkinitcpio] is an Arch Linux tool which builds an initrd based on
the config in `/etc/mkinitcpio.conf`

`mkinitcpio` has a concept of *hooks* are modules of (shell?) code that do
things both at build time and at boot time.  The main thing you do in
`/etc/mkinicpio.conf` is select which hooks to apply and their order.

[mkinitcpio]: https://wiki.archlinux.org/index.php/Mkinitcpio

This [Dm-crypt system configuration][dm-crypt-sysconfig] wiki suggests a value
for HOOKS, and I ended up with:

   HOOKS=(base udev keyboard modconf block encrypt lvm2 filesystems fsck)

Things to note are:

* `autodetect` is entirely missing from here.  This means all possible kernel
  modules etc will be included.  This is a silly thing to do, as `mkinicpio`
  generates just such a fallback image anyway.

* The `keyboard` hook is early in the list. Before the encrypt hook.  We
  need the keyboard up and running in order to unlock the disk.  Even if I had
  `autodetect`, I would probably put `keyboard` first, just to make sure I
  didn't have a hardware issue at this early stage.

Now we run

    mkinitcpio -p linux

Which dumps the files

    /boot/initramfs-linux-fallback.img
    /boot/initramfs-linux.img

### Booting directly from [UEFI](uefi)

We have already set `/boot` as the our EFI partition, and so the kernel and the

initramfs image are both already there.  And ArchLinux kernel's are built with
EFISTUB available, which means they are valid UEFI binaries.  So one way to
boot the kernel is:

* Boot the install stick, but select "EUFI Shell v2"
* Go to the EFI disk, i.e. type:

    fs1:

* Boot the kerenel, commad-line params and all

    \vmlinuz-linux cryptdevice=UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm root=LABEL=ROOT rw initrd=\initramfs-linux-fallback.img

Notice we use DOS style forward slashes for paths, even the initrd path.

**I made a tricky mistake with cryptdevice**.  I had left of the name
`:cryptlvm` from the end, and kept getting an error message instead of a
password prompt.  But because I didn't understand the messages, I believed for
a long time that my *keyboard* wasn't working and played around fruitlessly
with different keyboards, and different initrd configs.

I took a painfully long time to get the command line right.  Helpfully, UEFI
has it's own batch-files, so could iterate on the file:

        archlinux.nsh
        ------------------
        hd1b:
        \vmlinuz-linux cryptdevice=UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm root=LABEL=ROOT rw initrd=\initramfs-linux-fallback.img

* I believe `hd1b:` is the same device as `fs1`, but more with a more stable name.
* `UEFI` seems to prefer UCS-2 encoding, but you can use plain-old ascii files too.

### Setting up the boot menu

Once I had figured out how to boot the kernel, I shoved the command line
directly into the UEFI boot menu.  This can be done from the UEFI shell, but I
did it from the arch live-environment:

    DISK=/dev/nvme0n1
    PARTITION_NUM=1
    KERNEL_PARAMS='cryptdevice=UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm root=LABEL=ROOT rw initrd=\initramfs-linux-fallback.img'

    efibootmgr --create  \
            --disk "$DISK" \
            --part "$PARTITION_NUM" \
            --label "vmlinuz" \
            --loader "/vmlinuz-linux" \
            --unicode "$KERNEL_PARAMS"

I tried to make a boot entry that would run `archlinux.nsh`, but it never
worked, and I have no good debugging info about it.  My guess is that there is
no shell on the UEFI partition (I am running one that comes with the archlinux
boot disk).

Long term it would be nice to have both: use the shell-script version for
normal booting so there is a single-source-of-truth, and keep a conservative
one directly in the boot menu in case the shell isn't available.


Networking
----------

Once we reboot into the real system, all our lovely auto-detected networking
goes out the window.  Two options are documented here:

* [netctl](netctl) a profile based network manager from Arch that drops into
  systemd quite nicely -- i.e.  individual profiles are also systemd services.

* [nothing at all](netdiy) tools like `wpa_supplicant` and `dhcpd` are quite
  capable of auto-connecting you to the network.  The coordination can be
  supplied by systemd alone.

Wayland
-------

I am using [Wayland](wayland), via the *sway* compositor with the *kitty*
terminal emulator.

Microcode
---------

Following: https://wiki.archlinux.org/index.php/Microcode, arch makes it easy
to get manufacturer microcode.  [Here](microcode) is how we load it at boot.

[Suspend and Hibernate](suspend)
-----------------

Following
https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate.


TODO
----

* secure boot
* firmware
* Put autodetect back into the HOOK, but try uncompressed.
  * Can/should it go before udev (and if does that produce a smaller image?)
  * Can the keyboard hook go before udev?
