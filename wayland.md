WAYLAND
=======

I've decided to use [Wayland][arch-wayland] instead of Xorg.  The two most components that will
bit us are:

* A *compositor* this is the actual binary that runs the window system, but is also the
  window manager.

* An X11 server this is XWayland.  Once it is installed, the compositor will automatically
  use it behind the scenes to run X11 stuff.

The compositor I chose is tiling one called [sway][arch-sway].  They say it's like *i3*.

[arch-wayland]: https://wiki.archlinux.org/index.php/Wayland
[arch-sway]: https://wiki.archlinux.org/index.php/Sway

Sway
----

Get it

    pacman -S sway

Run it

    sway

And (on `scooter19`) it just worked.  The important key bindings are:

* _Win + ENTER_ launches a terminal.
* _Win + d_ brings up the interactive menu.
* _Win + e_ quits (subject to a nag dialog).

But:

* There is no menu
* There is no terminal

We need to add some packages, and probably also do some config.  For config:

    mkdir -p ~/.config/sway
    cp /etc/sway/config ~/.config/sway
    vi ~/.config/sway/config

XWayland
--------

So far, we have been using pure Wayland.  But eventually we will want some X11
program.  So let's be prepared:

    pacman -S \
        xorg-xrdb \
        xorg-server-xwayland 

(D)Menu
-------

The menu works by presenting a program of your choice in the status bar.  That program defaults
to `dmenu`, so:

    pacman -S dmenu

Terminals
---------

### urxvt 

The default terminal is `urxvt`, so 

    pacman -S rxvt-unicode

But the default config is black-on-white, and bad in other ways so we want

    ~/.Xresources
    ----------------
    URxvt*background: black
    URxvt*foreground: white
    . . . whatever else you want . . . 
	

But .Xresources doesn't just *work*. 
But X11 is a *network service* made by geniouses from MIT who anticipated the
way things work in Big Data nowdays.  It ain't gonna just *work*.  You have to
*deploy* the configs with

    xrdb .Xresources

But you don't want to do that manually every time.  You have to condigure the
deployment of configs.

So hack intol~/.config/sway/config and add

    . . .
    exec xrdb ~/.Xresources
    . . .

#### But urxvt is still terrible

As far as I know

* It can't change the font size interactively
* It can't render a (captial) "W" in italics.  It just draws a box.

### alacritty

This is an OpenGL based terminal that knows about Wayland (and also X).  It
claims to be the fastest terminal around -- but that's probably a lie. 

It works.  So install it with:

    pacman -S alacritty

And configure it with

    ~/.config/sway/config
    --------------------
    . . .
    # set $term urxvt
    set $term alacritty
    . . .

### kitty

`kitty`, like alacritty it is a new Wayland-aware terminal based on OpenGL.
Install it with:

    pacman -S alacritty

And configure it with

    ~/.config/sway/config
    --------------------
    . . .
    # set $term urxvt
    set $term kitty
    . . .

The main trick is that it does scrollback differently.  `SHIFT + Pg[Up|Down]'
don't work.  Instead press `CTRL+SHIFT H` and it will bring the scrollback
buffer up in `less` (or whatever your `$PAGER` is, I assume).
