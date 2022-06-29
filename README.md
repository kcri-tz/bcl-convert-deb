# bcl-convert-deb

This repository has instructions to generate a .deb file containing the
Illumina `bcl-convert` utility, for convenient installation on Debian and
Ubuntu systems.

**Note** see the bottom of this document for the issue with bcl-convert 3.10,
and why you may want to stay on 3.9.2.


## Introduction

`bcl-convert` comes only as an RPM package, downloadable from
<https://support.illumina.com/sequencing/sequencing_software/bcl-convert/downloads.html>
(requires login).

`bcl-convert` is just a single executable, so you could also unpack the RPM
and drop `bcl-convert` in your `~/bin` or `/usr/local/bin`.  (Why don't
Illumina distribute it as a tarball?)

However, one benefit of wrapping it in a deb is that we can add in a workaround
for `bcl-convert`'s mistaken assumption that it can log to `/var/log/bcl-convert`.
It can't and it shouldn't.

> Please someone tell the `bcl-convert` developers that `/var/log` is for
> system software and daemons, not for user programs.  Why not?  This would
> require a _world writeable_ directory: free for all unlimited storage on
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

The [User Guide](https://support-docs.illumina.com/SW/BCL_Convert/Content/SW/FrontPages/BCL_Convert.htm)
is perfectly devoid of information, but some documentation for bcl-convert can be gleaned from the
[bcl2fastq to bcl-convert upgrade guide](https://support.illumina.com/bulletins/2020/10/upgrading-from-bcl2fastq-to-bcl-convert.html),
and the [compatibility sheet](https://support.illumina.com/sequencing/sequencing_software/bcl-convert/compatibility.html),
found [hidden in the bottom drawer of a locked filing cabinet stuck in a disused lavatory with a sign on the door saying
"Beware of the Leopard"](http://www.goodreads.com/quotes/40705-but-the-plans-were-on-display-on-display-i-eventually).


#### Issue with bcl-convert 3.10

BCL-Convert 3.10 introduced a regression, namely erroring out when two of
your i7 or i5 indices differ 2nt or less.  The only workaround is to set
`BarcodeMismatchesIndex1,0` and `BarcodeMismatchesIndex2,0` in the sample
sheet, but this then discards every read that does not have two perfectly
matching indices.

The flawed logic behind this appears to be that, under the default allowed
mismatch (1nt), if you use barcodes `AAAA` and `AAGG` (different by 2nt),
`bcl-convert` should refuse to work because there are possible index reads
(in fact, precisely two: `AAAG` and `AAGA`) that are equidistant from
either barcode.

Huh, so what's the problem?  Just throw them in the `Undetermined` bin
where they belong (if they're even encountered).  Of the other 254 possible
reads, 240 are more than 1nt away from both (hence irrelevant), and the
remaining 14 are perfectly assignable because they are 2nt from one barcode,
but 0 or 1 from the other.

With 8nt barcodes, for those 2 rightly ambiguous ones, you're needlessly
discarding another 38 valid reads; with 10nt barcodes, 50.  All assuming
that you were willing to accept a 1nt mismatch in your index reads (which
you did because you set `BarcodeMismatchesIndex` to 1).

The bug appears to have been introduced with the new default of having the
mismatch threshold apply "or-or" across the i7 and i5 reads: reject when
_either_ exceeds it, whereas previously this happened only when _both_ did.
That change is fine, it's the added erroring out that is the issue.

