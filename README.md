# dpkg-deb-repo-scripts
Custom DEBIAN scripts for making deb packages and adding them to repos

I have noticed over the past few years that documentation for Debian customization and development has been hard to find, or easy to find and written poorly. This repository will hold the documentatation to help anyone interested in developing for the DEBIAN package manager, APT. 

If you are interested in developing your own distruction based off of DEBIAN and came up short of good documentation, please check my custom DEBIAN ISO Scripts repository for more information.

# DEB Files
This section will break down all files and what they require for building your packages for DEBIAN repositories.
## Control File
The control file contains the following information,

```Package: byteforce
Version: 1.1
Section: forensics
Priority: optional
Architecture: all
Depends: cmatrix
Maintainer: Douglas Berdeaux <weaknetlabs@gmail.com>
Description: Offline Digital Forensics Tool / Malware Analysis
```
Example outut from ByteForce in the WeakNet Labs code repository:
```
  wnl87:/pwnt/forensics# apt-cache search byteforce
  byteforce - Offline Digital Forensics Tool / Malware Analysis
```
When installing ByteForce, we see the dependency listed:
```
wnl87:/pwnt/forensics# apt-get install byteforce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  cmatrix
Suggested packages:
  cmatrix-xfont
The following NEW packages will be installed:
  byteforce cmatrix
0 upgraded, 2 newly installed, 0 to remove and 1 not upgraded.
Need to get 113 kB of archives.
After this operation, 33.8 kB of additional disk space will be used.
Do you want to continue? [Y/n]
```
We see "and one not upgraded" because I actually threw in there libpcap-dev which was already installed.
