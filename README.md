# Debian packaging for Xviewer plugins

This repository contains the Debian packaging for the Linux Mint
[Xviewer plugins](https://github.com/linuxmint/xviewer-plugins), using the
upstream **3.4.4** release. The source tarball is fetched from the upstream
release tag; upstream code is not copied into this packaging repository.

The package builds one binary package, `xviewer-plugins`. It installs the six
plugins built by upstream: Exif Display, Export to Folder, Map, Python Console,
Send by Mail, and Slideshow Shuffle. Upstream has disabled Postr because the
external `postr` program is no longer in Debian.

## Build on Debian

Build and install [xviewer](https://github.com/The-Specter-X/xviewer-deb)
first, including its `xviewer-dev` and `gir1.2-xviewer-3.0` packages. They
are not yet available from the official Debian archive. On the minimal build
VM, install the three matching locally built `.deb` files with `apt install`
so dependencies are resolved, then run:

```sh
git clone https://github.com/The-Specter-X/xviewer-plugins-deb.git
cd xviewer-plugins-deb
sudo apt install devscripts debhelper dh-python meson ninja-build
uscan --download-current-version --destdir ..
sudo apt-get build-dep .
dpkg-buildpackage -us -uc -b
```

`uscan` downloads and repacks the upstream release as
`../xviewer-plugins_3.4.4+ds.orig.tar.xz`. Alternatively, `gbp import-orig
--uscan` can import the upstream tarball into the Git branches defined in
`debian/gbp.conf`. The `-b` build creates a local binary package for testing;
use a full signed source build for a Debian upload.

Install the resulting `../xviewer-plugins_*.deb` on the separate Cinnamon VM
that has matching Xviewer packages. In Xviewer, open **Edit → Preferences →
Plugins** and check that the six plugins appear and can be enabled.

## Debian status

The existing WNPP request is [#830625](https://bugs.debian.org/830625). Its
current title is an RFP. Before a prospective Debian upload, the maintainer
can retitle and claim that existing report as an ITP, rather than opening a
second WNPP report.

`debian/salsa-ci.yml` uses the standard Salsa CI recipe. On Salsa, set the
project's CI configuration file path to `debian/salsa-ci.yml` after pushing
the repository. CI build dependencies cannot resolve `xviewer-dev` from the
official Debian archive until Xviewer has been uploaded there, or the pipeline
is configured to consume matching Xviewer packages from a trusted package
repository.
