Lenovo Thinkpad X1 Carbon 5th Gen
=================================


The [Arch Linux][arch] documentation for the [Thinkpad X1 Carbon 5th
Gen][arch-x1-5] laptops is at
https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Carbon_(Gen_5)

[arch]: https://www.archlinux.org
[arch-x1-5]: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Carbon_(Gen_5)

My particular laptop, which I call `scooter19` and is a late model in the
series:

    dmidecode -t system

Yields

    dmidecode 3.2
    Getting SMBIOS data from sysfs.
    SMBIOS 3.0.0 present.

    Handle 0x000C, DMI type 1, 27 bytes
    System Information
        Manufacturer: LENOVO
        Product Name: 20HRA04JAU
        Version: ThinkPad X1 Carbon 5th
        Serial Number: PF1KVS2D
        UUID: cd8556cc-3195-11b2-a85c-ce18f22d8072
        Wake-up Type: Power Switch
        SKU Number: LENOVO_MT_20HR_BU_Think_FM_ThinkPad X1 Carbon 5th
        Family: ThinkPad X1 Carbon 5th

Booting
-------

* `F1` gives us the BIOS config sceeen.
* `F12` gives the UEFI boot menu.  This is a standardised menu that can
  be edited using generic UEFI tools.

The machine came with [Secure Boot][arch-secure-boot] turned on, so before I
could boot the stick I had to turn it off from the BIOS config screen (`F1`).

[arch-secure-boot]: https://wiki.archlinux.org/index.php/Secure_Boot
