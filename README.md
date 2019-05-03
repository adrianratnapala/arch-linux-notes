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

    dd status=progress bs=8192000 if=archlinux-2019.04.01-x86_64.iso of=<b>/SOME USB DEVICE/</b>

