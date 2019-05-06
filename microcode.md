MICROCODE
=========

Following: https://wiki.archlinux.org/index.php/Microcode

Modern CPUs all have bugs.  Luckily they bugs are in the microcode, or are
worked around using microcode that can be loaded at boot.  The vendors ship the
microcode (somehow) and Arch Linux provides it as packges.  When you instlal
them, you get the microcode in the form of an initrd image in `/boot`.

The trick here is that you can "stack" multiple initrds, so you put the
microcode (loader) first, and the "real" initrd next.

Before doing anything:

<pre>
[raka@scooter19 arch-linux-notes]$ cat /sys/devices/system/cpu/cpu0/microcode/version 
<b>0x8e</b>
[raka@scooter19 arch-linux-notes]$ cat /sys/devices/system/cpu/cpu0/microcode/processor_flags 
0x80
[raka@scooter19 arch-linux-notes]$ grep microcode /proc/cpuinfo 
microcode       : <b>0x8e</b>
microcode       : <b>0x8e</b>
microcode       : <b>0x8e</b>
microcode       : <b>0x8e</b>
</pre>


Install the package
-------------------

    pacman -S intel-ucode 
    ls -l /boot/*-ucode.img

Hack the kernel params
----------------------

       DISK=/dev/nvme0n1
       PARTITION_NUM=1
       CRYPT_DEVICE="UUID=cf550c62-63b0-49fc-804a-3b4b112bcc03:cryptlvm"
       INITRD_LIST=(
		intel-ucode.img
		initramfs-linux-fallback.img
       )
       VMLINUZ_BOOTNUM=0
       
       _initrd=$(printf "initrd=\%s " "${INITRD_LIST[@]}")
       KERNEL_PARAMS="cryptdevice=$CRYPT_DEVICE root=LABEL=ROOT rw $_initrd"

       efibootmgr -v -B -b "$VMLINUZ_BOOTNUM" 
       efibootmgr -v -b "$VMLINUZ_BOOTNUM" \
               --disk "$DISK" \
               --part "$PARTITION_NUM" \
               --label "vmlinuz" \
               --loader "/vmlinuz-linux" \
               --unicode "$KERNEL_PARAMS"

Verify
----------------------

Once we reboot we can check that the microcode version changed.

<pre>
[raka@scooter19 arch-linux-notes]$  cat /sys/devices/system/cpu/cpu0/microcode/version 
<b>0x9a</b>
[raka@scooter19 arch-linux-notes]$ cat /sys/devices/system/cpu/cpu0/microcode/processor_flags 
0x80
[raka@scooter19 arch-linux-notes]$ grep microcode /proc/cpuinfo 
microcode       : <b>0x9a</b>
microcode       : <b>0x9a</b>
microcode       : <b>0x9a</b>
microcode       : <b>0x9a</b>
</pre>

