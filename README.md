# Xviewer-plugins packaging for Debian

This repository contains Debian packaging for [Linux Mint's xviewer-plugins](https://github.com/linuxmint/xviewer-plugins), targeting upstream tag **3.4.4** and source version **3.4.4-1**. The inspected tag points to commit `34a81b1c3de5b819994fbb8814ee8b162dc64d58`. Upstream code is downloaded separately.

The source builds `xviewer-plugins`, containing Exif Display, Export to Folder, Map, Python Console, Send by Mail, and Slideshow Shuffle. Postr remains disabled upstream. Debhelper generates debug symbol packages automatically.

## Build and check

Use a minimal Debian unstable build VM and a separate Debian Cinnamon desktop VM for interactive tests. Install `build-essential`, `devscripts`, `dpkg-dev`, `lintian`, `sbuild`, and `autopkgtest` on the build VM. Configure an unstable sbuild/schroot testbed before using the isolated commands below.

Build Xviewer first from [xviewer-deb](https://github.com/The-Specter-X/xviewer-deb). Install its three matching local packages (`xviewer`, `xviewer-dev`, and `gir1.2-xviewer-3.0`) with APT on the build VM. They are not yet in the official Debian archive.

From this packaging checkout:

```sh
uscan --download-current-version --destdir ..
mkdir ../xviewer-plugins-3.4.4
tar -xf ../xviewer-plugins_3.4.4.orig.tar.gz -C ../xviewer-plugins-3.4.4 --strip-components=1
# Replace the extracted Linux Mint packaging completely.
rm -rf ../xviewer-plugins-3.4.4/debian
cp -a debian ../xviewer-plugins-3.4.4/
cd ../xviewer-plugins-3.4.4
sudo apt build-dep .
dpkg-buildpackage -us -uc
lintian -i -I --pedantic ../xviewer-plugins_3.4.4-1_*.changes
sbuild -d unstable ../xviewer-plugins_3.4.4-1.dsc
autopkgtest ../xviewer-plugins_3.4.4-1_amd64.changes -- schroot unstable-amd64-sbuild
```

The `mkdir` deliberately fails if the build tree already exists; start with a fresh tree for each source preparation. Replace `amd64` and the testbed name with your configured architecture and schroot.

Use the original upstream archive, without a `+ds` repack just to remove `debian/`. Source format `3.0 (quilt)` replaces that directory when extracting the Debian source package. This packaging-only repository does not assume imported upstream or pristine-tar branches. `debian/watch` discovers numbered upstream tags; inspect and update the changelog before packaging a newer release.

The installed-package test loads all six plugins through libpeas and executes code in the Python Console. Also enable and exercise each plugin in Xviewer on the desktop VM.

Install the resulting runtime packages with APT on the separate Cinnamon VM. Test opening, saving where applicable, help, printing, thumbnails and plugins, including Wayland and X11 sessions where available. Automated smoke checks do not cover all interactive behavior.

## Salsa and submission

The intended Salsa project is `https://salsa.debian.org/Overseer/xviewer-plugins`. Push the packaging history there and set the CI configuration path to `debian/salsa-ci.yml` under **Settings → CI/CD → General pipelines**. The standard Salsa recipe is retained.

The existing WNPP request is [#830625](https://bugs.debian.org/830625). Claim it as an ITP before requesting sponsorship.

Keep the changelog `UNRELEASED` during preparation. After clean Debian unstable builds, installed-package tests, desktop checks and copyright review pass, finalize it for `unstable`, build and sign a source upload on the machine holding your signing key, and upload it to mentors.debian.net for sponsor review. GitHub commits and Salsa CI do not upload to Debian. Do not commit binaries; any test binary release should include the matching source, `.changes`, `.buildinfo` and checksums.

An isolated build or Salsa pipeline also needs the matching Xviewer build dependencies from a trusted local package repository until Xviewer enters Debian. Installing them on the host alone does not make them available inside a clean testbed.
