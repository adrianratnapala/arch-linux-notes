UEFI
====

UEFI is the modern replacement for BIOS: *i.e.* it boots the OS off disk and
provides some firmware to initalise or manage the hardware.  But it's better to
think of it as the return of MS-DOS.  After all, DOS has itself been called a
glorified boot loader.  There is a mapping like this


        1995                            2019
        ----                            ----
        BIOS                            Firmware (coreboot, or proprietry)
        MS-DOS                          UEFI
        DOS/4GW, Win95, SYSLINUX        Operating systems & boot loaders

MS-DOS and UEFI don't do multitasking or resource protection but they both
provide a simple file-system and can run programs.  Some of those programs are
native, using only the facilities of DOS or UEFI, such programs include
interactive menus, a command line shell and memory-resident drivers.  Other
programs are OS kernels that take over the system, or else bootloaders for such
kernels.

The point about this is that while BIOS and such firmware is fairly
machine-specific (and frequently dodgy), the UEFI is layer is pretty well
accessible and can be worked on with generic tools.

The UEFI equivalent of `C:\` is called the EFI partition (TODO: does it always
turn as `FS1:\` in the shell?).  It is a Fat32 partition, and generally the
first one on the disk.

* **TODO**: where is the UEFI "kernel" stored?  Is it in nvram or is there some
  equivalent to MSDOS.SYS

## The Boot Menu

One big difference is that the *contents* of the boot menu is at a standardised
location (**TODO** in NVRAM?).  The menu is basically a mapping from textual
menu entries to UEFI command-lines to run.  Your machine's shipped version of
UEFI will boot a default item from this menu, or else present the user a
menu-selection UI.

You can edit the menu using the UEFI tools available from the UEFI shell.  But
there is also a unix tool `efibootmgr` that ships with the Arch Linux live ISO.
So you can set up a menu entry with commands like:


       DISK=/dev/nvme0n1
       PARTITION_NUM=1
       KERNEL_PARAMS='cryptdevice=UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm root=LABEL=ROOT rw initrd=\initramfs-linux-fallback.img'

       efibootmgr --create  \
               --disk "$DISK" \
               --part "$PARTITION_NUM" \
               --label "vmlinuz" \
               --loader "/vmlinuz-linux" \
               --unicode "$KERNEL_PARAMS"

##  Installing a shell
----------------------

[shell-spec]: https://uefi.org/sites/default/files/resources/UEFI_Shell_Spec_2_0.pdf

The [UEFI shell][shell-spec] implementation is part of an open-source effort
called TianoCore, but we (semi) install it through an AUR package.

    pacman -S fakeroot

    cd ~/aur
    git clone https://aur.archlinux.org/uefi-shell-git.git
    cd uefi-shell-git
    makepkg -si

Which will ask for your password to become root, and then do all sorts of
stuff, ending with

    . . .
    UEFI Shell v2 binaries are installed at /usr/share/uefi-shell/*.efi

Which is a fairly annoying place to keep things.  To make the shell useable we
have to put it on the EFI partition.

   cp /usr/share/uefi-shell/shellx64_v2.efi /boot/EFI/Boot/
   cp /usr/share/uefi-shell/shellx64_v2.efi /boot/EFI/Boot/shell.efi

And then use `efibootmgr`

    pacman -S efibootmgr

to tell add a boot entry for launching the shell.

    DISK=/dev/nvme0n1
    PARTITION_NUM=1


    efibootmgr --create  \
            --disk "$DISK" \
            --part "$PARTITION_NUM" \
            --label "UEFI shell" \
            --loader "/EFI/Boot/shell.efi"

    # clear the boot order (i.e. go in numerical order).
    # efibootmgr -O


APPENDIX: boot/set-up-efi-boot.sh
---------------------------------
<code>
DISK=/dev/nvme0n1
PARTITION_NUM=1
CRYPT_DEVICE="UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm"
INITRD_LIST=(
        intel-ucode.img
        initramfs-linux-fallback.img
)
KERNEL_PARAM_LIST=(
        "cryptdevice=$CRYPT_DEVICE"
        "root=LABEL=ROOT"
        "resume=/dev/cryptvg/SWAP"
        rw 
        $(printf " initrd=\%s" "${INITRD_LIST[@]}")
)
VMLINUZ_BOOTNUM=0

KERNEL_PARAMS=$(printf " %s" "${KERNEL_PARAM_LIST[@]}")

efibootmgr -B -b "$VMLINUZ_BOOTNUM" 
efibootmgr -v -b "$VMLINUZ_BOOTNUM" --create \
       --disk "$DISK" \
       --part "$PARTITION_NUM" \
       --label "vmlinuz" \
       --loader "/vmlinuz-linux" \
       --unicode "$KERNEL_PARAMS"
</code>
