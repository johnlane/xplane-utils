
X-Plane Utilities
=================

This is a collection of utilities and notes to ease the
set-up and running of X-Plane in a multi-headed configuration.

(this file is formatted with [MarkDown](http://www.showdown.im))

John Lane

Utilities
---------

A few utilities to ease setup and running of X-Plane.

### xpslave

`xpslave` will create one or more slave copies of X-Plane
configured to act as additional visuals for an already existing
master copy. Usage:

    $ xpslave master [-f] slave1...slaven

If the `-f` parameter is given then any existing slave directories
are replaced.

The command:

    $ xpslave master-centre slave-{left,right}

will create slaves suitable for use by xpstart, described next.

### xpstart

`xpstart` launches a new multi-headed X-Plane session. It launches
a new X-Server (it assumes an appropriate xorg.conf with sufficient
 X-screens configured).

The X-Plane copies are assumed to be called `master-centre`, `slave-left`
and `slave-right`.

### xpget

Use `xpget` to get installers, documentation and other useful files from
the internet, where available, from an inserted X-Plane DVD.

Usage:

    $ xpget [cdrom-device] [x-plane-version]

Examples:

  * Default, internet get for version 10

        $ xpget

  * DVD + internet get for version 10

        $ xpget /dev/sr0

  * internet get for version 9

        $ xpget 9

  * DVD + internet get for version 9

        $ xpget /dev/sr0 9

### xpiso

`xpiso` copies a DVD into an iso image, stores and names it appropriately
for use by the other scripts.

Usage:

    $ xpiso [cdrom-device]

### Other Files

The xplane-utils also includes:

  * this file (README.md);
  * the MIT LICENSE;
  * jhat, a pre-compiled (i686) executable of jhat;
    (sources: <http://github.com/fugalh/jhat.git>. GNU GPL v2)
  * documentation and sample files, see below.

Documentation
-------------

*What do you want? Blood?*

For anyone interested in more details,  I have written *One Man's Way
to Configure X-Plane* which explains my set-up and how my utilitles
work. It's in the *doc* directory along with my xorg configuration
files.

(Read `doc/OneMansWayToConfigureXPlane.md` for more information.)

Usage
-----

(Assuming a 32-bit environment)

Get `xplane-utils`:

    $ git clone git://github.com/johnlane/xplane-utils X-Plane
    $ cd X-Plane

Get stuff

    $ ./xpget 9

Run the Installer

    $ ./"X-Plane Demo Installer Linux"

Move and rename the install: from `~` to `master-centre`:

    $ mv ~/"X-Plane 9 Demo" master_centre

Run the demo copy (in `master-centre`) and make sure it works. Be happy,
buy the full version.

If you have the full version, put DVD#1 in your drive and make an iso copy
of it (needed for run-time validation):

    $ ./xpiso

This will create a file called `iso/xplane_9_dvd1.iso`.

Set up multi-head: create slaves.

    $ ./xpslave master-centre slave-{left,right}

Run:

    $ ./xpstart

Scope
-----

I created these utilities for my own specific purposes and their suitability 
for use in any other scenario (i.e. by anyone other than me, or on any system
other than my own) is unknown. Anyone who thinks they may be useful is, however,
welcome to give them a try but don't complain if they don't work for you!

Should anyone make improvements they are more than welcome to send me a pull
request on GitHub.

License
-------

MIT License: See LICENSE file

Latest Version
--------------

Obtain the latest version from [GitHub](https://github.com/johnlane/xplane-utils).
