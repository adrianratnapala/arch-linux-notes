NETCTL
======

Following https://wiki.archlinux.org/index.php/Netctl

[arch-netctl]: https://wiki.archlinux.org/index.php/Netctl

### Wired

Wired is easy

    /etc/netctl/digitech.dhcp
    -------------------------
    Description='A basic dhcp ethernet connection'
    Connection=ethernet
    Interface=enp60s0u2u4
    IP=dhcp

Then launch it with

    netctl start digitech.dhcp


### Wireless

This is still a work in progress, so far I have

<pre>
Description='Automatically generated profile by wifi-menu'
Interface=wlp4s0
Connection=wireless
Security=wpa
ESSID=TP-Link_EDF8
IP=dhcp
Key=<b>MY WIFI KEY</b>
<b>WPADriver=wext</b>
</pre>

*WPADriver* was a stumbling block.  By default it tries `nl80211`, which failed
(I think there is a bug in the way it uses `libnetlink`).


