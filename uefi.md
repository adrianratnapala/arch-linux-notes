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


