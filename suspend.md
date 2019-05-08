SUSPEND AND HIBERNATE
=====================

Following: https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate

This is al based on `scooter19` which is a (5th Gen Thinkpad X1)(thinkpad-x1-5th).

Suspend to RAM
--------------

This seems to work right out of the box.  I press `systemd suspend` and the
thing goes to sleep until I press the power button, then it is bavck.  Same if
I press `Fn+4`.  Open questions are:

* Make it work upon closing the lid.

* How to add hooks: e.g. lock the screen first. 

### Close the lid

By default, nothing happens when you close and open the lid.   But *systemd*
can do it for you if you edit (uncomment a line in) a suprisingly old simple
and old fashioned config:

    /etc/systemd/logind.conf
    -----------------------
    ...
    HandleLidSwitch=suspend 
    ...

It seems you have to restart a daemon to make this take:

    systemctl restart systemd-logind

But `sway` crashes when I do this, even if I do it from separate VT.

You just edit the file, no need to load the config.  Oddly, the comment at the
top claims that this is a compiled in default, but still I have to uncomment
it.

See also:
* https://wiki.archlinux.org/index.php/Power_management

#### Does it work?

I'm pretty sure it does, and `HandleLidSwitch=suspend` makes it suspend on
close and restore on open.  But the restore process is so quick that it's not
obvious that it really worked.  But at least the screen is blank for a second
or so -- unlike when I leave it commented.  So at least I know the setting does
*something*.


### Locking the screen.

My favourite screen locker is `physlock`

    pacman -S physlock

It blocks out all VTs but its own, upon which it displays a textual password
prompt.

The state of being suspended (or hibernated?) seems to be modeled in *systemd*
by a thing called a `sleep.target`.  You can define a systemd service that
hooks into transitions to and from the state represented by a target.

For physlock we create a weird hybrid user/root thing.

    /etc/systemd/system/suspend@.service
    -------------------------------------
    [Unit]
    Description=Lock all the things.
    Before=sleep.target

    [Service]
    User=%I
    Type=forking
    ExecStart=/usr/bin/physlock -ds

    [Install]
    WantedBy=sleep.target

Ending the name in `@` is special magic, because we can enable it as:

    systemctl enable suspend@raka
    -------------------------------------
    Created symlink /etc/systemd/system/sleep.target.wants/suspend@raka.service → /etc/systemd/system/suspend@.service.

And now things Work For Me.  I don't know exactly what the semantics of the substitutions
and the username is.

Hibernate to Disk
-----------------

### MkSwap

I left an LVM logical volume called SWAP, specifically for hibernation -- not
because I want swap in ordinary usage.

    [root@scooter19 raka]# lvdisplay 
      --- Logical volume ---
      LV Path                /dev/cryptvg/SWAP
      LV Name                SWAP
      VG Name                cryptvg
      LV UUID                xGFaR7-QABf-SouL-I25W-u1Cd-x7zm-ut5qnJ
      LV Write Access        read/write
      LV Creation host, time archiso, 2019-04-28 16:33:28 +1000
      LV Status              available
      # open                 0
      LV Size                20.00 GiB
      Current LE             5120
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           254:1

(N.B. This is is a UUID with real SouL!).

So make that swap

      mkswap /dev/cryptvg/SWAP -L SWAP

The Arch documentation is not very clear, but I am guessing you need
to actually *use* the swap: so we add:

    /etc/fstab
    ----------
    LABEL=SWAP none swap defaults  0 0

### Configure the boot

#### Kernel

Do some black magic to ensure that `resume=/dev/cryptvg/SWAP` appears in the
kernel params.  For me that means editing _/boot/set-up-efi-boot.sh_.

#### *initrd*

Add the _resume_ hook to the *mkinitcpio.conf*; the principle is you add it
after whatever you need to make the device appear.  For a swap *file* that
would be *filesystem*.  For us it is *lvm2*.  Thus

    /etc/mkinitcpio.conf
    --------------------------
    . . . 
    HOOKS=(base udev keyboard modconf block encrypt lvm2 resume filesystems fsck)
    . . . 

And put it:

    mkinitcpio -p linux

Now we test it with

    systemctl hibernate 

And it works!  So let's upgrade the lid-close function

    HandleLidSwitch=hibernate
    /etc/systemd/logind.conf
    -----------------------
    ...
    HandleLidSwitch=suspend-then-hibernate
    ...


`suspend-then-hibernate` does what it says on the tin, it suspends the machine
but then hibernates after 180 minutes.  This is configured in
`/etc/systemd/sleep.conf`
