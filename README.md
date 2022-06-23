# bcl-convert-deb

This repository has instructions to generate a .deb file containing the
Illumina `bcl-convert` utility, for convenient installation on Debian and
Ubuntu systems.


## Introduction

`bcl-convert` comes only as an RPM package, downloadable from
<https://support.illumina.com/sequencing/sequencing_software/bcl-convert/downloads.html>
(requires login).

`bcl-convert` is just a single executable, so you could also unpack the RPM
and drop `bcl-convert` in your `~/bin` or `/usr/local/bin`.  (In fact, it is
beyond me why Illumina don't just distribute it as a tarball.)

However, one benefit of wrapping it in a deb is that we can add in a workaround
for `bcl-convert`'s mistaken assumption that it can log to `/var/log/bcl-convert`.
It can't and it shouldn't.

> Please someone tell the `bcl-convert` developers that `/var/log` is for
> system software and daemons, not for user programs.  Why not?  This would
> require a _world writeable_ directory, free for all unlimited storage on
> the system log partition.  What could possibly go wrong?

`bcl-convert` has no option to log elsewhere, so the workaround is to make
`/var/log/bcl-convert` a symlink to `/tmp`, and we can patch that into the
deb.

In case you already installed `bcl-convert`, to fix the `/var/log` bungle:

    sudo rm -rf /var/log/bcl-convert &&
    sudo ln -sfT /tmp /var/log/bcl-convert


## Creating the deb

Download the latest BCL-Convert (requires login) from
<https://support.illumina.com/sequencing/sequencing_software/bcl-convert/downloads.html>

Install the Ubuntu `alien` and `fakeroot` packages

    sudo apt install alien fakeroot

Convert the `rpm` to `deb` (ignore the script warnings)

    fakeroot alien --fixperms --bump=0 bcl-convert-*.rpm

Unpack the `deb`

    dpkg-deb -R bcl-convert_*.deb unpack

Patch the /var/log botch

    rmdir unpack/var/log/bcl-convert
    ln -sfT /tmp unpack/var/log/bcl-convert

Repack the `deb`

    fakeroot dpkg-deb -b unpack bcl-convert_*.deb

Done


## Install the deb

As usual

    sudo apt install ./bcl-convert_*.deb


## Running

Run as follows:

    bcl-convert --bcl-input-directory INDIR --output-directory OUTDIR

Note the input and output directories are mandatory as you'd expect, not
options as this suggests.  (Why not standard `bcl-convert INDIR OUTDIR`?)

