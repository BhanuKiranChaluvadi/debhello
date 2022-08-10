- [Option 1. Create debian using debuild - debhello](#option-1-create-debian-using-debuild---debhello)
  - [STEP-1: Run Docker](#step-1-run-docker)
  - [STEP-2: Create templated Debian files](#step-2-create-templated-debian-files)
    - [Debmake](#debmake)
  - [STEP-3: Edit Debian files](#step-3-edit-debian-files)
    - [debian/rules](#debianrules)
    - [debian/control](#debiancontrol)
    - [debian/copyright](#debiancopyright)
  - [STEP-4: Build Debian Pacakage](#step-4-build-debian-pacakage)
    - [Debuild](#debuild)
- [Option 2. Create debian using dpkg-deb](#option-2-create-debian-using-dpkg-deb)
- [Inspect Debian Package](#inspect-debian-package)
- [Unpack and Edit Debian Package](#unpack-and-edit-debian-package)
- [Useful links](#useful-links)
# Option 1. Create debian using debuild - debhello
We are closely following the (tutorial)[https://www.debian.org/doc/manuals/debmake-doc/ch04.en.html] from debian.org and we will be using docker container for this setup. 
## STEP-1: Run Docker
Initialize docker container
```bash
docker run -it --rm --user vscode mcr.microsoft.com/vscode/devcontainers/base:bullseye
```
Install necessary dependencies inside container
```bash
sudo apt update
sudo apt install -y debhelper debmake nano tree
```

## STEP-2: Create templated Debian files
### Debmake
* The debmake command is the helper script for the Debian packaging. It creates good template files in debian folder.
```bash
mkdir ~/git && cd ~/git
git clone https://github.com/BhanuKiranChaluvadi/debhello.git
cd debhello
debmake -t -p debhello -u 0.1 -r 1 -x4
cd ..
ls
tree debhello-0.1
```
## STEP-3: Edit Debian files
Modify file to look like these
### debian/rules
```bash
 $ nano debhello-0.1/debian/rules
 ... hack, hack, hack, ...
 $ cat debhello-0.1/debian/rules
#!/usr/bin/make -f
export DH_VERBOSE = 1
export DEB_BUILD_MAINT_OPTIONS = hardening=+all
export DEB_CFLAGS_MAINT_APPEND  = -Wall -pedantic
export DEB_LDFLAGS_MAINT_APPEND = -Wl,--as-needed

%:
        dh $@

override_dh_auto_install:
        dh_auto_install -- prefix=/usr
```
### debian/control
```bash
 $ nano debhello-0.1/debian/control
 ... hack, hack, hack, ...
 $ cat debhello-0.1/debian/control
Source: debhello
Section: devel
Priority: optional
Maintainer: Osamu Aoki <osamu@debian.org>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.5.1
Homepage: https://salsa.debian.org/debian/debmake-doc
Rules-Requires-Root: no

Package: debhello
Architecture: any
Multi-Arch: foreign
Depends: ${misc:Depends}, ${shlibs:Depends}
Description: Simple packaging example for debmake
 This Debian binary package is an example package.
 (This is an example only)
```
### debian/copyright
```bash
 $ nano debhello-0.1/debian/copyright
 ... hack, hack, hack, ...
 $ cat debhello-0.1/debian/copyright
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: debhello
Upstream-Contact: Osamu Aoki <osamu@debian.org>
Source: https://salsa.debian.org/debian/debmake-doc

Files:     *
Copyright: 2015-2021 Osamu Aoki <osamu@debian.org>
License:   Expat
 Permission is hereby granted, free of charge, to any person obtaining a
 copy of this software and associated documentation files (the "Software"),
 to deal in the Software without restriction, including without limitation
 the rights to use, copy, modify, merge, publish, distribute, sublicense,
 and/or sell copies of the Software, and to permit persons to whom the
 Software is furnished to do so, subject to the following conditions:
 .
 The above copyright notice and this permission notice shall be included
 in all copies or substantial portions of the Software.
 .
 THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
 OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
 MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
 IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
 CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
 TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
 SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

## STEP-4: Build Debian Pacakage
### Debuild
* The **debian/rules** file defines how the Debian binary package is built.
* The **debuild** command is a wrapper script of the dpkg-buildpackage command to build the Debian binary package under the proper environment variables.

```bash
cd ~/git/debhello-0.1
debuild
cd ..
ls
```

# Option 2. Create debian using dpkg-deb
Steps were taken from [youtube tutorial](https://www.youtube.com/watch?v=ep88vVfzDAo&ab_channel=JoeCollins)
```bash
# 0. export some variables
# example export deb_path=hello_0.1-1_amd64.deb
export deb_path=<path>/<pkg_name>_<version>_<architecture>
# 1. create a directory to hold the project
mkdir -p deb_path
# 2. create a directory called "DEBIAN" inside the project
mkdir -p deb_path/DEBIAN
# 3. copy files into project root directory and include the final paths
# /usr/bin/ would be <deb_path>/usr/bin/
# /opt/ would <deb_path>/opt/
# 4. Create a control file in DEBIAN:
touch deb_path/DEBIAN/control
# 5. Now add the necessary meta data to the control file:
Package: my-program
Version: 1.0
Architecture: all
Essential: no
Priority: optional
Depends: packages;my-program;needs;to;run
Maintainer: Your Name
Description: A short description of my-program that will be displayed when the package is being selected for installation. 
# 6. If desired a "preinst" and/or "postinst" script can be added that execute before and/or after installation. They must be given proper execute permissions to run:
# Add commands you'd like to run in postinst and then set the permissions to 755.
touch deb_path/DEBIAN/postinst
# 7. generate package
dpkg-deb --build deb_path
```

# Inspect Debian Package
```bash
mkdir -p ~/git/inspect
cp ~/git/debhello_0.1-1_amd64.deb ~/git/inspect/
cd ~/git/inspect
dpkg-deb --help
dpkg-deb --info ./debhello_0.1-1_amd64.deb
dpkg-deb --contents ./debhello_0.1-1_amd64.deb
dpkg-deb --extract ./debhello_0.1-1_amd64.deb .
ls
tree ./usr
./usr/bin/hello
```

# Unpack and Edit Debian Package
```bash
mkdir -p ~/git/edit
cp ~/git/debhello_0.1-1_amd64.deb ~/git/edit/
cd ~/git/edit
```
```bash
# unpack debian
$ ar -x ./debhello_0.1-1_amd64.deb
$ ls
# lets edit control file. Unpack control.tar.xz
$ tar -xvf control.tar.xz
$ ls
$ nano control
... hack, hack, hack, ...
... Added tree as a Dependency ...
... Do what ever suits you need ...
# Edit a depenency
$ cat control
Package: debhello
Version: 0.1-1
Architecture: amd64
Maintainer: Bhanu Kiran <bkch@universal-robots.com>
Installed-Size: 30
Depends: libc6 (>= 2.2.5), tree
Section: devel
Priority: optional
Multi-Arch: foreign
Homepage: <insert the upstream URL, if relevant>
Description: auto-generated package by debmake
 This Debian binary package was auto-generated by the
 debmake(1) command provided by the debmake package.

# Repack control.tar.xz - specific order has to be followed 
# $ tar -Jvcf control.tar.xz {post,pre}{inst,rm}  conffiles control
$ tar -Jvcf control.tar.xz {post,pre}{inst,rm}  control

# Repack debhello_0.1-1_amd64.deb - specific order has to be followed 
$ ar rcs debhello_0.1-1_amd64.deb debian-binary control.tar.xz data.tar.xz

# inspect
$ dpkg-deb --info ./debhello_0.1-1_amd64.deb
```
# Useful links 
* [Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/index.en.html)
* [Chapter 4. Simple Example](https://www.debian.org/doc/manuals/debmake-doc/ch04.en.html)
* [Chapter 8. More Examples](https://www.debian.org/doc/manuals/debmake-doc/ch08.en.html)
* [debian c progam example](https://github.com/MichinariNukazawa/debian_package_c_application_automate_example)
* [junyelee blogspot](http://junyelee.blogspot.com/2021/06/build-debian-packages.html)
* [HowToPackageForDebian](https://wiki.debian.org/HowToPackageForDebian)
