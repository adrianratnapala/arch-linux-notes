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

Once you the boot menu from the stick, select **Boot Arch Linux**.  This will
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

Now that you are on the net, there is probably a working SSHD, so it is
worth:

* SSHing in from another machine (especially because it lets you cut and
  paste to and from the terminal).

* Starting a `screen` session.

## Check for UEFI

These instructions assume the system supports [UEFI](uefi) this is the case
for both `scoter19` and `gollum`.  You can test it by seeing if the EFI NVRAM
has any variables in it:

```
ls /sys/firmware/efi/efivars  | head -n 10                                          :(
AMD_PBS_SETUP-a339d746-f678-49b3-9fc7-54ce0f9df226
AMITSESetup-c811fa38-42c8-4579-a9bb-60e94eddfb34
AmdSetup-3a997502-647a-4c82-998e-52ef9486a247
AuditMode-8be4df61-93ca-11d2-aa0d-00e098032b8c
Boot0001-8be4df61-93ca-11d2-aa0d-00e098032b8c
Boot0002-8be4df61-93ca-11d2-aa0d-00e098032b8c
Boot0003-8be4df61-93ca-11d2-aa0d-00e098032b8c
Boot0004-8be4df61-93ca-11d2-aa0d-00e098032b8c
BootCurrent-8be4df61-93ca-11d2-aa0d-00e098032b8c
BootOptionSupport-8be4df61-93ca-11d2-aa0d-00e098032b8c
```

(That was on `gollum`).


Disks
-----

I want to encrypt my as much as possible, but still boot simply.  So
for both systems I have:

* An unencrypted `/boot`, this is the same thing as the [(U)EFI
  parition](uefi).
  * On `scooter19` this is 256 MiB; it shipped with the machine and is
    labelled `SYSTEM`.
  * On `gollum` this is 1GiB, I created it myself and it is labeled `EFI`.
* Put the rest of the disk on encrypted [LUKS][luks] volume `cryptlvm`
* Map this 1-1 onto a [LVM][lvm] volume group `cryptvg`, on which I have
  - A 50G volume  volume called `ROOT` (`/`, `/var` and `/usr`)
  - A 20G volume  volume called `SWAP`, not to used, but reserved for when I
    do hibernate-to-disk
  - The rest is `HOME`, for `/home`

(TODO: it looks like `scooter19` is (wrongly) using swap!)

The main authority for the encryption stuff is [https://wiki.archlinux.org/index.php/Dm-crypt][luks].  Which has a sub-page
for this kind of [LVM-on-LUKS][lvm-on-luks] setup.

The details below are for `scooter19`, the details for gollum are in
[2020-02-gollum.md](2020-02-gollum.md).


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
which mirrors the mount on the final system.

On scooter19 that meant:

    mount /dev/cryptvg/ROOT /mnt -L ROOT
    mkdir /mnt/home
    mount /dev/cryptvg/HOME /mnt/home -L HOME
    mount /dev/by-label
    mkdir /mnt/boot
    mount /dev/disk/by-label/SYSTEM efi # a.k.a /dev/nvme0n1p1

On Gollum

```
mount /dev/cryptvg/ROOT /mnt
mkdir /mnt/home
mkdir /mnt/boot
mount /dev/cryptvg/HOME /mnt/home
mount /dev/disk/by-label/EFI /mnt/boot
```


### Installing the packages

Now we will install base packages, downloading them from repositories defined
in `/etc/pacman.d/mirrorlist`.  You can edit that file to use mirrors near
you, see:
https://wiki.archlinux.org/index.php/Installation_guide#Select_the_mirrors

At this point it really is best to `ssh` in from another machine, and to have
a screen session.

Now we install quite a complete set of stuff:

    pacstrap /mnt base linux linux-firmware vim lvm2 netctl


### Post-package setup

    genfstab -L /mnt >> /mnt/etc/fstab
    arch-chroot /mnt

Sets us up in some approximation of the real system (`-L` to genfstab uses
label-based names, `-U` uses UUIDs).

There are many other straightforward instructions to follow at
https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system

Here is what I did for `gollum` -- though the only thing that should be
different is the hostname.

```
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Australia/Sydney /etc/localtime
hwclock --systohc

/usr/bin/vim locale.gen
```

And we uncomment `en_AU` lines ONLY.  It's important to not allow `en_US` a
look-in.

```
locale-gen
echo "LANG=en_AU.UTF-8" > /etc/locale.conf

echo "gollum" > /etc/hostname

cat > /etc/hosts <<EOF
127.0.0.1 localhost
::1       localhost

192.168.1.19    scooter19.home       scooter19       # 7C:2A:31:E5:39:59
192.168.1.11    br-hl-3170cdw.home   br-hl-3170cdw   # 3C:2A:F4:3C:85:87
192.168.1.11    BRN3C2AF43C8587.home BRN3C2AF43C8587 # 3C:2A:F4:3C:85:87
192.168.1.10    gollum.home          gollum          # 96:C9:95:11:FB:96
192.168.1.12    ravana.home          ravana          # 70:54:D2:44:EB:11
EOF
```



Booting
-------

There are many options, and I have chosen:

* **Use busybox initrd** (the default), as opposed to systemd.

* **Using [EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB)**.  Kernels
  (including the `vmlinuz` that Arch hands us) are compiled so that they double
  as standard EUFI executables that can be run from its shell, or directly from
  the boot menu.

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

`scooter19`:
   HOOKS=(base udev keyboard modconf block encrypt lvm2 filesystems fsck)

`gollum`:
   HOOKS=(base udev keyboard autodetect modconf block encrypt lvm2 resume filesystems fsck)

Things to note are:

* Dropping `autodetect` from `scooter19`  means all possible kernel modules
  etc will be included.  This is a silly thing to do, as `mkinicpio` generates
  just such a fallback image anyway.

* The `keyboard` hook is early in the list. Before the encrypt hook.  We
  need the keyboard up and running in order to unlock the disk.  Even if I had
  `autodetect`, I would probably put `keyboard` first, just to make sure I
  didn't have a hardware issue at this early stage.

Now we run

    mkinitcpio -p linux

This means run mkinitcpio getting flags from a "preset" called `linux`.

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
