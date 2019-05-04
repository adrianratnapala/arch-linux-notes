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
[aarnet.edu.au][arch-at-aarnet]

    curl -O https://mirror.aarnet.edu.au/pub/archlinux/iso/2019.04.01/archlinux-2019.04.01-x86_64.iso
    curl -O https://mirror.aarnet.edu.au/pub/archlinux/iso/2019.04.01/archlinux-2019.04.01-x86_64.iso.sig

[archdl]: https://www.archlinux.org/download/
[arch-at-aarnet]: https://mirror.aarnet.edu.au/pub/archlinux/iso/

### Verify that ISO

I don't like or understand GPG enough to know what to do with the `.sig` file, except that since I am
doing this on an existing arch installation, I run.

    pacman-key -v archlinux-2019.04.01-x86_64.iso.sig

This checks the `.sig` against the corresponding file (in the same directory) and says:

<pre>
==> Checking archlinux-2019.04.01-x86_64.iso.sig...
gpg: assuming signed data in 'archlinux-2019.04.01-x86_64.iso'
gpg: Signature made Tue 02 Apr 2019 02:31:03 AEDT
gpg:                using RSA key <b>4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC</b>
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
dd status=progress bs=8192000 if=archlinux-2019.04.01-x86_64.iso of=<b>/SOME USB DEVICE/</b>
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

I want to encrypt my as much as possible, but still boot simply.  So I will set
up `scooter19` with:

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

Because the `/boot` partition is shared outside of Linux, I have not formatted
it.  That made it a nice place to keep logs of what I was doing.  The following
is not exactly a shell script, but I could uncomment whichever line I wanted
and then run it:

### Formatting etc.

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


TODO
----

* microcode
* X11
* secure boot
* firmware
* hibernate to disk
