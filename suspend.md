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

You just edit the file, no need to load the config.  Oddly, the comment at the
top claims that this is a compiled in default, but still I hace to uncomment
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

