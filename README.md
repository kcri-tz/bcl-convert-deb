# bcl-convert

The Illumina BCL-Convert software only comes as an RPM package.
Use the Debian `alien` command to convert it.

## Steps

* Download the latest BCL-Convert from <https://support.illumina.com/sequencing/sequencing_software/bcl-convert/downloads.html>
* Run `fakeroot alien --fixperms --bump=0 bcl-convert-*.rpm`
* Proceed as normal: `apt install ./bcl-convert_*.deb`
* Set ownership on log directory: `chown root:adm /var/log/bcl-convert`
* Set permissions on log directory: `chmod 0775 /var/log/bcl-convert`
* Try run `bcl-convert --help` to test

