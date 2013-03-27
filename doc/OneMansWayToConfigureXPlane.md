
One man's way to Configure X-Plane
==================================

As the saying goes, *there are many ways to skin a cat*. Likewise, I am sure there 
are many ways to configure a multi headed X-Plane. These notes describe one way,
the way that works for me.

I share this here in the hope that some might find it useful, This is what works
for me. To coin another phrase: "your mileage may vary".

For the impatient: just read the *Usage Summary* at the end of this file.

(this file is formatted with [MarkDown](http://www.showdown.im))

John Lane, October 2012.

*Updated March 2013 to incorporate the 64-bit version of X-Plane 10.*

Background
----------

My machine is a Core-i7 920 running at 4.0Ghz, with 12GB of RAM.

My system configuration is virtualised: I have a minimal install on the metal and
then employ Linux Containers (LXC) to run virtualised hosts for various purposes.
One of those purposes is to run X-Plane.

I run Arch Linux which is a 64-bit operating system. To use 32-bit applications,
I run a 32-bit chroot and I use "schroot" to launch 32-bit applications as an
unprivileged user.

My desktop is therefore running inside a 64 bit container and that container has
a 32-bit chroot for running 32 bit applications.

None of the above is critical to running X-Plane excpet that you need a 32-bit
environment of some sort. You now know what I have and that it works for me nicely.

*I now run X-Plane 10 as a 64-bit application from within my 64-bit desktop container.
I no longer need to run it in a 32-bit chroot (although I can, and it still works).*

X-Org configuration
-------------------

I run a triple-headed system for my day-to-day work. I have an NVidia GTX670 GPU 
driving two 1600x1200 (4:3) displays either side of a central 1920x1200 (16:10)
display. I run this with TwinView and it works well for me.

Twinview is not ideal for a multi-headed X-Plane: To do multi-head on X-Plane you
need to run a separate copy of X-Plane for each physical display, each of which
needs to be a separate X-screen.

Rather than mess around with my general desktop configuration, I have taken the
approach to run a completely separate X-Server on a different virtual terminal for
X-Plane. This is configured as three separate X-Screens: optimal for running three
full-screen copies of X-Plane.

Because X-Plane is in a separate chroot, I can configure X-Org inside that chroot
independently of my dekstop. My X-Org configuration consists of two files:

  * `/etc/X11/xorg.conf.d/10-input.conf`
  * `/etc/X11/xorg.conf.d/21-nvidia-settings.conf`

(copies of these files are included in `doc/xorg`).

To run X-Plane without a chroot but in an environment where another X-Org 
configuration is in use, the X-Org configuration files for X-Plane's dedicated
X-Server can be placed in a separate subdirectory `/etc/X11/xorg.xplane.conf.d`
and the X-Server can be instructed to use that configuration directory instead
of the default.

When I run X-Plane, I first launch a new X-Server on a separate vt. Because the
initiation of the X-Server automatically switches the current vt to the one where
that X-Server is running, this is transparent. It remains transparent when that
X-Server exits because the vt is automatically switched back to the one it was on
previously. At least that's what happens for me. You can also switch back and
forth by using the `<CONTROL>+<ALT>+<FnKey>` key combination at any time.

To launch an X-Server on a separate vt:

    $ xinit <program> -- :<display> <vt>

where `<display>` is the X-display and `<vt>` is the vt number. I keep these the same,
but that isn't necessary. I use vt12, so:

    $ xinit <program> -- :12 vt12

The `<program>` is whatever you want the server to run. In my case it is a bash
script that starts X-Plane but I'll come back to that later. To use a custom X-Org
configuration, append a `-configdir` argument, for example:

    $ xinit <program> -- :12 vt12 -configdir xorg.xplane.conf.d

### Mouse Cursor

I had an issue that, sometimes, I would not see the mouse cursor when X-Plane
started.  This, I believe, is due to the X-root pointer not being initialised
because I don't run a window manager (I am running X-Plane full screen so there
is no need for one).

To resolve this problem, I explicitly set a cursor on each of the three
X-Screens.  I use this command:

    $ xsetroot -cursor_name left_ptr

It has to be done once for each screen (i.e set DISPLAY=12:0 for the first one, 
issue the command, set DISPLAY=12:1 then do it again, and so on).

The cursor name `left_ptr` is one of the stock cursors included in X-Org. You can
see all the available stock cursors [here](http://tronche.com/gui/x/xlib/appendix/b).

(Gratitude to those helping me on the X-Plane forum with this one)

X-Plane Configuration
---------------------

I took guidance from [here](http://forums.x-plane.org/index.php?showtopic=44217).
I configured two new "slaves" off a working "master" copy of X-Plane using links
to avoid duplicating storage. I wrote a bash script called `xpslave` to do this
for me.  I won't repeat the steps here: look at my script or the above linked
forum posting.

I experimented with writing X-Plane configuration files from scratch and found that
it was sufficient to write only those options that I needed (perhaps I am wrong
here but it does seem to work ok: I discovered the options through experimentation).
They are plain-text files.

The master configuration files exist already; the slave ones do not so I create
them.  They contain a 2-line header and I copy this from the master file to the
slaves. I write two files:

  * `X-Plane.prf`
  * `Screen Res.prf`

(that latter is called "X-Plane Screen Res.prf" in X-Plane 10)

I set the master to full-screen (I modify the existing "Screen Res.prf" file,
changing the option `_is_full_ALL` to 1. I write a similar value to the same file
for each slave.

All other options are in the "X-Plane.prf" file. I modify the master and create
the slave copies.

I enable the frame rate display ON (this is optional and can be skipped) by
setting `_Breal0` to 1.

I read the master's UDP ports from its "X-Plane.prf" file. I then assign different
ports for each slave. The master, by default, uses 49000 and 49001. I chose to
assign slave ports incrementally starting with 49010. The first slave gets 49010
and 49011. The second one gets 49012 and 49013.

I set each slave's ports by writing `_rcv_port` and `_snd_port` values to their
files.

I set a visual offset for each slave: the left one being -45 degrees and the right
one being +45 degrees. This is written to `_off_lat_deg`.

I disable sound on the slaves because, otherwise, all three copies would emit the
same sounds (which is somewhat pointless).

For X-Plane-10, I also set a visual field of view for each slave. This is 45 degrees
for my 4:3 monitors. It is 60 degrees for a widescreen one (as per the X-Plane-10
manual). This is written to `_FOVx`.

Input Devices
-------------

I have a Saitek yoke and pedal set. Because I run inside a container, I don't have
the benefit of udev to set them up so I therefore needed to cater for it manually.
At least it means I understand what is going on, and I now understand that version
10 does it differently to version 9.

I look in `/sys/class/input` to find my devices. Version 9 uses `js` devices whereas
`event` devices are used by version 10. I look for devices with the word `Yoke` or
`Pedals` in their names (as, for example, read from `/sys/class/input/js0/device/name`).

I check for the presence of matching `/dev/input` devices and, if missing, create them
(using the information from, for example, `/sys/class/input/js0/dev`).

*Note:* if using `systemd` inside an LXC container the above device node setup will
need to be done inside an autodev hook. This is described [here](https://wiki.archlinux.org/index.php/Lxc-systemd).

### Hat Switch

To make my hat switch work, I fire up a copy of the ever-useful `jhat` utility with
the details of my yoke. To work out the correct axes I used `jstest` (in the
`joyutils` package downloaded from the Arch Linux AUR).

### Plug-In order

I have noticed that the order that the input devices are connected matters to
X-Plane. I have to plug in my Yoke first and then my Pedals otherwise they map
incorrectly to the X-Plane functions.

Start-Up Script
---------------

My start-up script is called `xpstart` and it does the following:

It first checks to see if the X-Server is running. If it isn't then it launches
`xinit`, citing itself as the program for xinit to launch. The script then exits.

Xinit starts the X-Server and then launches the set-up script again. This time
the script:

  * determines the X-Plane version (it checks the header in X-Plane.prf);
  * looks for an iso image of the X-Plane DVD#1 and, if found, mounts it on /mnt;
  * determines the system architecture so that it can start the correct executable;
  * locates input devices and ensures device nodes exist;
  * launches jhat (in the background), if available, for the yoke;
  * for each screen:
    * sets the X-Pointer with xsetroot;
    * starts X-Plane in background, adding its PID to a wait list;
  * waits for the wait-list PIDS to end ;
  * kills off jhat, if running;
  * unmounts any dvd image.

##### Note

I wait for all copies of X-Plane to exit. This is so you can do `File->Quit` on
each one to close them cleanly. If you don't close X-Plane-10 cleanly it
complains on the next launch about crashing the time before. Originally with
X-Plane-9, I waited only for the master coppy of X-Plane and didn't worry about
the others.

Other Things
------------

I wrote a quick and dirty utility called `xpget` that downloads stuff from 
X-Plane.com and other places (the demo, updated, manual, jhat, etc).

I have another useful script on [github](https://github.com/johnlane/isomounter)
called `isomounter` that allows you to run another application (such as the X-Plane
installer), while presenting a list of available iso images and enabling these to
be mounted as required by the application. I did this to ease the installation of
scenery: with `isomounter` I can easily switch between DVD1 and any other scenery
disc that the installer may ask for.

Usage Examples
--------------

### Full Version 10 installation

Doing the below with the X-Plane 10 DVD 1 in the drive results in a
triple-headed configuration of X-Plane-10 full version (for me,
on my system. *YMMV*):

    $ git clone git://github.com/johnlane/xplane-utils X-Plane-10
    $ cd X-Plane-10
    $ ./xpget /dev/sr0
    $ ./xpiso
    $ run32 isomounter "X-Plane 10 Installer Linux"
      (select DVD 1 and install to default location)
    $ mv ~/"X-Plane 10" master-centre
    $ run32 master-centre/X-Plane-i686
      (Verify it works, and then exit. This writes master configs)
    $ ./xpslave master-centre slave-{left,right}
    $ run32 xpstart 

### Web Demo Version 10 installation

Doing the below results in a triple-headed configuration of the X-Plane-10 web 
demo (for me, on my system. *YMMV*):

    $ git clone git://github.com/johnlane/xplane-utils X-Plane-10
    $ cd X-Plane-10
    $ ./xpget
    $ run32 "X-Plane 10 Demo Installer Linux"
    $ mv ~/"X-Plane 10 Demo" master-centre
    $ run32 master-centre/X-Plane-i686
      (Verify it works, and then exit. This writes master configs)
    $ ./xpslave master-centre slave-{left,right}
    $ run32 xpstart 

### Web Demo Version 9 installation

And doing the below results in a triple-headed configuration of the X-Plane-9 web 
demo (for me, on my system. *YMMV*):

    $ git clone git://github.com/johnlane/xplane-utils X-Plane-9
    $ cd X-Plane-9
    $ ./xpget 9
    $ run32 "X-Plane Demo Installer Linux"
    $ mv ~/"X-Plane 9 Demo" master-centre
    $ run32 master-centre/X-Plane-i686
      (Verify it works, and then exit. This writes master configs)
    $ ./xpslave master-centre slave-{left,right}
    $ run32 xpstart 

(`run32` is an alias that launches my 32-bit chroot: `schroot -p --`)

END. JL 041212
