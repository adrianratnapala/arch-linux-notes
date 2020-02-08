DIY (ish) network
=================

While I like [netctl](netctl), `netctl-auto` borked auto-connection of WiFi.
And it seems the basic tools *[wpa_supplicant][arch-wpas]* and
*[dhcpcd][arch-dhpcpd]* are actualy very capable.  Better stll,
[systemd][arch-systemd] is a good way of gluing things like this together.


[arch-dhcpcd]: https://wiki.archlinux.org/index.php/Dhcpcd
[arch-systemd]: https://wiki.archlinux.org/index.php/systemd
[arch-wpasupp]: https://wiki.archlinux.org/index.php/WPA_supplicant

Manually
--------

WPA-Supplicant logs onto WiFi.  You provide credentials for each network, and
it will automatically pick one and switch as things change.  And then
(dhcpd)[arch-dhcpd] deals with the consequences; but that's a story for later.

    /etc/wpa_supplicant/wpa_supplicant.conf
    ---------------------------------------
    network={
	    ssid="TP-Link_EDF8"
	    psk="....."
    }
    network={
	    ssid="raka-nokia"
	    psk="....."
    }

And we can launch it with

     wpa_supplicant -i wlp4s0 -D wext \
	-c /etc/wpa_supplicant/wpa_supplicant.conf \
	-d |& tee wpa-raka-nokia.log

* `-d` debug outpu (to stderr).
* `-c` config file (I think there is no default).
* `-i` network interface.
* `-D` the "driver" -- names the code in `wpa_supplicant` for talking to the
  kernel. On Linux the options are
  - `nl80211` is the new, recommended one, uses netlink,
  - `wext` is the one that works.

There is some bug that forces us to use wext, which is not the default.  So
when we start automating this, we need to figure out how to sneak the `-D`
option in.

This works, but merely connecting to the WiFi means we are stuck at Level 2.
We need to "Connect to the Internet".  That is, get to level 3.
    
    dhcpd wlp4s0 

systemd
-------

### `wpa_supplicant.service`

    systemctl enable wpa_supplicant@wlp4s0.service
    Created symlink /etc/systemd/system/multi-user.target.wants/wpa_supplicant@wlp4s0.service → /usr/lib/systemd/system/wpa_supplicant@.service

But this default config doesn't do the magic we want.  So we type

    systemctl edit wpa_supplicant@wlp4s0.service

Which opens

    /etc/systemd/system/wpa_supplicant@wlp4s0.service.d/override.conf
    --------------------------------------------------------------------
    [Service]
    ExecStart=
    ExecStart=/usr/bin/wpa_supplicant -Dwext -c/etc/wpa_supplicant/wpa_supplicant.conf -i%I 

Note the empty `ExecStart=`, this is special-syntax which allows us to force
override an existing value.

    # systemctl status wpa_supplicant@wlp4s0.service
    ● wpa_supplicant@wlp4s0.service - WPA supplicant daemon (interface-specific version)
       Loaded: loaded (/usr/lib/systemd/system/wpa_supplicant@.service; enabled; vendor preset: disabled)
      Drop-In: /etc/systemd/system/wpa_supplicant@wlp4s0.service.d
	       └─override.conf
       Active: active (running) since Fri 2019-05-10 20:22:32 AEST; 2min 40s ago
     Main PID: 20408 (wpa_supplicant)
	Tasks: 1 (limit: 4915)
       Memory: 1.0M
       CGroup: /system.slice/system-wpa_supplicant.slice/wpa_supplicant@wlp4s0.service
	       └─20408 /usr/bin/wpa_supplicant -Dwext -c/etc/wpa_supplicant/wpa_supplicant.conf -iwlp4s0

    May 10 20:22:32 scooter19 systemd[1]: Started WPA supplicant daemon (interface-specific version).
    May 10 20:22:32 scooter19 wpa_supplicant[20408]: Successfully initialized wpa_supplicant
    May 10 20:22:32 scooter19 wpa_supplicant[20408]: ioctl[SIOCSIWENCODEEXT]: Invalid argument
    May 10 20:22:34 scooter19 wpa_supplicant[20408]: wlp4s0: Trying to associate with ac:84:c6:3e:ed:f8 (SSID='TP-Link_EDF8' freq=2412 MHz)
    May 10 20:22:34 scooter19 wpa_supplicant[20408]: wlp4s0: Associated with ac:84:c6:3e:ed:f8
    May 10 20:22:34 scooter19 wpa_supplicant[20408]: wlp4s0: WPA: Key negotiation completed with ac:84:c6:3e:ed:f8 [PTK=CCMP GTK=TKIP]
    May 10 20:22:34 scooter19 wpa_supplicant[20408]: wlp4s0: CTRL-EVENT-CONNECTED - Connection to ac:84:c6:3e:ed:f8 completed [id=0 id_str=]
    [root@scooter19 wpa_supplicant@wlp4s0.service.d]# 


### `dhcpcd.service`
