# MOVED

Important notice; I will not be maintaining this repo.
This PKGBUILD will be kept and updated at [my archlinux PKGBUILD repo](https://github.com/bbaserdem/Arch/tree/master/matlab).
I noticed a few people have this repo starred; so for integrating versions
later than 2020a with pacman; refer there instead.

# MATLAB: Archlinux integration

This PKGBUILD creates an Arch Linux package for MATLAB
Additionally; it also builds python integration and establishes jupyter kernel.

# Requirements

Besides the dependencies; the source files must be present at the directory.
The user must supply;

* **matlab.fik**: Plain text file installation key
* **matlab.lic**: License file
* **matlab.tar**: Software tarball

Then run `makepkg -s` from the PKGBUILD directory.
You probably want to run this locally; since the package will be huge.
Turn off compression if this package will only be locally installed;
this cuts a fair amount of time from the build.

## File Installation Key & Licence File:

[Here is the current link to the instructions](https://www.mathworks.com/help/install/ug/install-using-a-file-installation-key.html)

File installation key identifies this specific installation of matlab.
The license file authorizes that this key can use the toolboxes.
Follow the steps;

* Go to [License center](https://www.mathworks.com/licensecenter) on mathworks
* On install and activate tab; select (or create) an appropriate license
* Navigate to download the license file and the file installation key
* Download the **license file** and put the file in the repository
* Copy and paste the **file installation** key in a plain text file

## Tarball

**To run the installer, libselinux is needed!**

* [Download the matlab installer](https://www.mathworks.com/downloads)
* Unpack and launch the installer
* After logging in and accepting license; select
`Advanced Options > I want to download without installing`
from the top dropdown menu.
* Select the toolboxes you want in the `PKGBUILD`.
(Alternatively install them all)
* Download the files to an empty directory called `matlab`
* After downloading; from the parent directory; do
`tar --create --verbose --file matlab.tar.gz .../matlab`
to create the tarball.

# Tips

## Large files
To transport large files in fat32 media; use split and cat;
* **Split**: `split --bytes 3G --numeric-suffixes=0 matlab.tar.gz matlab.tar.gz.`
* **Concatenate**: `cat matlab.tar.gz.* > matlab.tar.gz`

## Modules

Matlab comes with a [lot of products](https://www.mathworks.com/products.html).
Most are not needed; check PKGBUILD to pick and choose appropriately.
