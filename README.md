# bcl-convert

The Illumina BCL-Convert software only comes as an RPM package.
Use the Debian `alien` command to convert it, then install normally.

## Installation Steps

* Download the latest BCL-Convert (requires login at Illumina), from
  <https://support.illumina.com/sequencing/sequencing_software/bcl-convert/downloads.html>
* Install the Ubuntu `alien` package: `sudo apt install alien`
* Run `fakeroot alien --fixperms --bump=0 bcl-convert-*.rpm` to generate deb file.
* Install the deb file: `sudo apt install ./bcl-convert_*.deb`
* Remove the log directory: `sudo rm /var/log/bcl-convert` (because only root can write into it)
* And turn it into a symlink to `/tmp`: `ln -sfT /tmp /var/log/bcl-convert`, so logs go
  to `/tmp/dragen_run_*` and will be cleaned up automatically
* Try run `bcl-convert --help` to test (and note the 0 byte length file in `/tmp`)

## Running bcl-convert

Run as follows (yes, it's non POSIX, how typical):

    bcl-convert --bcl-input-directory IN --output-directory OUT --no-lane-splitting

