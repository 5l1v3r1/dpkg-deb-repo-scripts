# dpkg-deb-repo-scripts
Custom DEBIAN scripts for making deb packages and adding them to repos

This repository will hold the documentatation to help anyone interested in developing for the DEBIAN package manager, APT. 

If you are interested in developing your own distribution (ISO) based off of DEBIAN and came up short of good documentation, please check my custom DEBIAN ISO Scripts repository for more information.

# Setup and Architecture of the Repository
I have created the directory, ```/web/weaknetlabs.com/repos/``` to host all of the repository stuff for WeakNet LINUX distributions.
## GnuPG Keys
Then I created a GPG key to sign all of my packages with to verify the identity and integrity of them by my users. This process will create a public and private key for us. We will only use the public key from this point fwd.
```
trevelyn@weaknetlabs:~$ gpg --gen-key
```
I did this with the user/UID that will be handling all of the packages development.
## Apache Rules (Privacy)
Next, I create a file ```/etc/apache2/conf.d/repos``` with the following contents. (You will have to update it to your own personal workspace)
```
<Directory /web/weaknetlabs.com/repos/ >
        # We want the user to be able to browse the directory manually
        Options Indexes FollowSymLinks Multiviews
        Order allow,deny
        Allow from all
</Directory>

# This syntax supports several repositories, e.g. one for Debian, one for Ubuntu.
# Replace * with debian, if you intend to support one distribution only.
<Directory "/web/weaknetlabs.com/repos/apt/*/db/">
        Order deny,allow
        Deny from all
</Directory>

<Directory "/web/weaknetlabs.com/repos/apt/*/conf/">
        Order deny,allow
        Deny from all
</Directory>

<Directory "/web/weaknetlabs.com/repos/apt/*/incoming/">
        Order allow,deny
        Deny from all
</Directory>
```
Run a test to make sure that the Apache2 configuration works and reload it in production.
```
trevelyn@weaknetlabs:~$ sudo apache2ctl configtest
[sudo] password for trevelyn:
Syntax OK
trevelyn@weaknetlabs:~$ sudo /etc/init.d/apache2 reload
[ ok ] Reloading web server config: apache2.
trevelyn@weaknetlabs:~$
```
## Configuration Directory in Repository
In the /web/weaknetlabs.com/repos/ directory, I created a conf directory like so,
```
trevelyn@weaknetlabs:~$ mkdir -p /web/weaknetlabs.com/repos/apt/wnl/conf
```
Remember, this is up to you to design and build out. The ```repos``` direcory is just my own personal preference. Next, I made a ```distributions``` file in the newly made conf directory with the contest like so:
```
Origin: WeakNet LINUX
Label: WeakNet LINUX
Codename: caffeine
Architectures: i386 amd64
Components: main
Description: Apt repository for WeakNet LINUX Packages
SignWith: <key-id>
```
Use the sub key-id (without the preceding length and fwd slash) from the output of the following command,
```
trevelyn@weaknetlabs:~$ gpg --list-keys
/home/trevelyn/.gnupg/pubring.gpg
---------------------------------
pub   2048R/EXXXXXXX 2011-05-09
uid                  Douglas Berdeaux <WeakNetLabs@Gmail.com>
sub   2048R/DXXXXXXX 2011-05-09

trevelyn@weaknetlabs:~$
```
Just use the DXXXXXXX portion of the "sub" public key for the SignWith entry.
## Options File
And finally, add an options file in the ewly created ```conf``` directory with the following contents.
```
verbose
basedir /web/weaknetlabs.com/repos/apt/wnl
ask-passphrase

```
Update the base directory above to point to your own, obiously.
# DEB Files
This section will break down all files and what they require for building your packages for DEBIAN repositories.

The first thisng that you want to do when creating a DEBIAN package is to create a working directory whre you will have each of your debian packages in subdirectories. For instance. 
```
WORKSPACE
 |->ByteForce
     |->files for ByteForce
 |->WARCARRIER
     |->files for WARCARRIER
```
I will simply be using /pwnt/forensics/ for my examples using WeakNet LINUX. For this, I will have to create a new directory tree in the WORKSPACE/ByteForce directory as so,
```
WORKSPACE
 |->ByteForce
     |->pwnt
         |->forensics
             |->ByteForce
                 |->ByteForce
```
This way, when the installation begins, the directory ```/pwnt/forensics/ByteForce``` will contain the ByteForce executable. This is just my preference as WeakNet LINUX contains all pentesting tools in ```/pwnt``` This means that for me to call the ByteForce application, I will need to be in the ```/pwnt/forensics/ByteForce``` directory. Here is an example of the ```/pwnt``` diretory as of WeakNet LINUX version 8.7:

```
wnl87:/pwnt# ls
80211              fuzzing                 redteam
80215              ids-ips                 reverse-engineering
active-directory   intelligence-gathering  social-engineering
audit-reporting    mitm                    toolbox
av                 networking              tor
av-bypass          OSINT                   versioning
databases          passwords               windows
denial-of-service  pentest-resources       wireless
exploitation       post-exploitation       www
forensics          privacy
wnl87:/pwnt# 

```

Next, we need to make a new directory that we will build a package in. E.g.: the "ByteForce" and "WARRCARRIER" directories listed above. I am going to do so using a git clone command, but you don't have to.

```
wnl87:/pwnt/forensics# git clone https://github.com/weaknetlabs/ByteForce
Cloning into 'ByteForce'...
remote: Counting objects: 78, done.
remote: Total 78 (delta 0), reused 0 (delta 0), pack-reused 78
Unpacking objects: 100% (78/78), done.
Checking connectivity... done.
wnl87:/pwnt/forensics#
```
This created the ByteForce directory for me. Next, we will clone THIS repository to have for reference purposes. Do not cd into the working directory.
```
wnl87:/pwnt/forensics# git clone https://github.com/weaknetlabs/dpkg-deb-repo-scripts/
Cloning into 'dpkg-deb-repo-scripts'...
remote: Counting objects: 21, done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 21 (delta 3), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (21/21), done.
Checking connectivity... done.
wnl87:/pwnt/forensics#
```
Now, we are left with the structure,
```
/pwnt/forensics/
 |->dpkg-deb-repo-scripts
     |->DEBIAN
         |->config.example
         |->control.example
 |->ByteForce
     |->files for ByteForce
```
Now, we just copy the entore DEBIAN directory from ./dpkg-deb-repo-scripts into ByteForce,
```
wnl87:/pwnt/forensics# cp -R dpkg-deb-repo-scripts/DEBIAN/ ByteForce/
wnl87:/pwnt/forensics#
```

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

## Preinst Script
This section describes the shell script that runs pre-installation. This file needs to the correct file permissions set. The file requires a ```chmod +x DEBIAN/preinst```

This script will run *BEFORE* the installation process occurs. These scripts are good for setting up your user's envionment for the application that you are installing.

## Postinst Script
This section describes the shell script that runs pre-installation. This file needs to the correct file permissions set. The file requires a ```chmod +x DEBIAN/postinst```

This script will run *AFTER* the installation process occurs. These scripts are good for cleaning up your user's envionment after the application installation. 

# Compiling the .DEB Package
Before continuing, please make sure the directory structure mathes that as in the above section. Now, just fill out the example files and rename them. Compile ByteForce and make the deb package like so.
(note, we can use "dpkg-depcheck -d ./configure" if there is a configuration file to build out dependency list)
```
wnl87:/pwnt/forensics# cd ByteForce/
wnl87:/pwnt/forensics/ByteForce# make
gcc byteforce.c -o ByteForce
wnl87:/pwnt/forensics/ByteForce# vim DEBIAN/control.example 
wnl87:/pwnt/forensics/ByteForce# vim DEBIAN/config.example 
wnl87:/pwnt/forensics/ByteForce# mv DEBIAN/control.example DEBIAN/control
wnl87:/pwnt/forensics/ByteForce# mv DEBIAN/config.example DEBIAN/config
wnl87:/pwnt/forensics/ByteForce# cd ..
wnl87:/pwnt/forensics# dpkg-deb --build ByteForce/ byteforce.deb
dpkg-deb: warning: 'ByteForce//DEBIAN/control' contains user-defined Priority value '0'
dpkg-deb: warning: ignoring 1 warning about the control file(s)
dpkg-deb: building package `byteforce' in `byteforce.deb'.
wnl87:/pwnt/forensics# ls byteforce.deb 
byteforce.deb
wnl87:/pwnt/forensics#
```
And we have our byteforce.deb file. Next, we need to add it to our repository using Reprepro.
# Reprepro
This section will describe how to add the package to a private repository.
## Add the .deb Package to the Repository
To add the debian package, or update one, we simply use the following command,
```
trevelyn@weaknetlabs:/web/weaknetlabs.com/repos/apt/wnl$ reprepro -P 0 -V -S forensics -C main includedeb caffeine byteforce.deb
Exporting indices...
 looking for changes in 'caffeine|main|i386'...
  replacing '/var/www/weaknetlabs.com/repos/apt/wnl/dists/caffeine/main/binary-i386/Packages' (uncompressed,gzipped)
 looking for changes in 'caffeine|main|amd64'...
  replacing '/var/www/weaknetlabs.com/repos/apt/wnl/dists/caffeine/main/binary-amd64/Packages' (uncompressed,gzipped)
* Douglas Berdeaux <WeakNetLabs@Gmail.com> needs a passphrase
Please enter passphrase:
Successfully created '/var/www/weaknetlabs.com/repos/apt/wnl/dists/caffeine/Release.gpg.new'
* Douglas Berdeaux <WeakNetLabs@Gmail.com> needs a passphrase
Please enter passphrase:
Successfully created '/var/www/weaknetlabs.com/repos/apt/wnl/dists/caffeine/InRelease.new'
Deleting files no longer referenced...
deleting and forgetting pool/main/b/byteforce/byteforce_1.1_all.deb
trevelyn@weaknetlabs:/web/weaknetlabs.com/repos/apt/wnl$
```
Easy, huh? 
## Check for Package Using "apt-cache search"
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

# Apt-Get Test
We need to add our gpg key from our server to the machine(s) that we want to install our software on to. 
```
root@wt87:~# wget -O - https://weaknetlabs.com/repos/apt/wnl/wnlrepo.gpg.key|apt-key add -
```
Add the repository to the /etc/apt/sources.list file,
```
# WeakNet LABS DEBIAN Repo
deb http://weaknetlabs.com/repos/apt/wnl caffeine main
```
Then update the database to include our cache and install the package using `apt-get`.
```
wnl87:~# apt-get update
Ign http://weaknetlabs.com caffeine/main Translation-en_US        
Ign http://weaknetlabs.com caffeine/main Translation-en
Fetched 498 kB in 3s (142 kB/s)
Reading package lists... Done
wnl87:~# apt-cache search byteforce
byteforce - Offline Digital Forensics Tool / Malware Analysis
wnl87:~# apt-get install byteforce -y
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  byteforce
0 upgraded, 1 newly installed, 0 to remove and 1 not upgraded.
Need to get 0 B/96.8 kB of archives.
After this operation, 0 B of additional disk space will be used.
Selecting previously unselected package byteforce.
(Reading database ... 166697 files and directories currently installed.)
Preparing to unpack .../archives/byteforce_1.1_all.deb ...
Unpacking byteforce (1.1) ...
Setting up byteforce (1.1) ...
wnl87:~# 
```
# Updating Packages
In this section I will cover howe to push updated packages to your production DEBIAN package repository.

If you make chnages to an application, you need to update the ```Version``` string in the DEBIAN/control file. Once done, creat ethe deb file using
```
dpkg-deb --build ByteForce/ byteforce.deb
```
Just as we did in the previous section. Remembner to note you path and working directory. Then, put the DEB file into your repository and run the ```reprepro``` command as we did the previous section. Afterwards, if you try to do an ```apt-get install byteforce``` you will see that there is no update until you run ```apt-get update```.
example:
```
wnl87:/pwnt/forensics# apt-get install byteforce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
byteforce is already the newest version.
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
wnl87:/pwnt/forensics# apt-get update
Ign http://weaknetlabs.com caffeine/main Translation-en_US
Ign http://weaknetlabs.com caffeine/main Translation-en
Fetched 30.2 kB in 2s (13.0 kB/s)
Reading package lists... Done
wnl87:/pwnt/forensics# apt-get install byteforce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  byteforce
1 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
Need to get 96.9 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 http://weaknetlabs.com/repos/apt/wnl/ caffeine/main byteforce all 1.4.0 [96.9 kB]
Fetched 96.9 kB in 0s (218 kB/s)
Reading changelogs... Done
(Reading database ... 166883 files and directories currently installed.)
Preparing to unpack .../byteforce_1.4.0_all.deb ...
This script ran BEFORE installation
Unpacking byteforce (1.4.0) over (1.2) ...
Setting up byteforce (1.4.0) ...
This ran AFTER installation
wnl87:/pwnt/forensics# 
```
# Summary
This should be enough to get us up and running with a simple private Debain APT/DEB package repository.
