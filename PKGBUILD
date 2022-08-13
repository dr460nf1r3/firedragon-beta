# Maintainer: dr460nf1r3 <dr460nf1r3 at garudalinux dot org>
# Contributor: torvic9 AT mailbox DOT org
# Contributor: lsf

pkgname=firedragon-beta-znver2
_pkgname=FireDragon
__pkgname=firedragon
pkgver=103.0.2
pkgrel=1
pkgdesc="Librewolf fork build using custom branding, settings & KDE patches by OpenSUSE"
arch=(x86_64 x86_64_v3 aarch64)
backup=('usr/lib/firedragon/firedragon.cfg'
        'usr/lib/firedragon/distribution/policies.json')
conflicts=(firedragon)
provides=(firedragon)
license=(MPL GPL LGPL)
url=https://gitlab.com/dr460nf1r3/settings/
depends=(gtk3 libxt mime-types dbus-glib ffmpeg nss ttf-font libpulse kfiredragonhelper
         aom icu pipewire dav1d)
makedepends=(unzip zip diffutils yasm mesa imake inetutils python-pyqt5 python-cmd2
             rust xorg-server-xwayland xorg-server-xvfb mold pciutils
             autoconf2.13 clang llvm jack nodejs cbindgen nasm
             python-setuptools python-zstandard git binutils lld dump_syms
             'wasi-compiler-rt>13' 'wasi-libc>=1:0+258+30094b6' 'wasi-libc++>13' 'wasi-libc++abi>13')
optdepends=('firejail-git: Sandboxing the browser using the included profiles'
            'profile-sync-daemon: Load the browser profile into RAM'
            'whoogle: Searching the web using a locally running Whoogle instance'
            'searx: Searching the web using a locally running searX instance'
            'networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech'
            'hunspell-en_US: Spell checking, American English'
            'libappindicator-gtk3: Global menu support for GTK apps'
            'appmenu-gtk-module-git: Appmenu for GTK only'
            'plasma5-applets-window-appmenu: Appmenu for Plasma only')
options=(!emptydirs !makeflags !strip !lto !debug)
install=$__pkgname.install
source=(https://archive.mozilla.org/pub/firefox/releases/"$pkgver"/source/firefox-"$pkgver".source.tar.xz{,.asc}
        "$__pkgname.desktop"
        "git+https://gitlab.com/dr460nf1r3/common.git"
        "git+https://gitlab.com/dr460nf1r3/settings.git"
        "librewolf-source::git+https://gitlab.com/librewolf-community/browser/source.git"
        "librewolf-settings::git+https://gitlab.com/librewolf-community/settings.git"
        "cachyos-source::git+https://github.com/CachyOS/CachyOS-Browser-Common.git")
# source_aarch64=()
sha256sums=('766183e8e39c17a84305a85da3237919ffaeb018c6c9d97a7324aea51bd453aa'
            'SKIP'
            '158152bdb9ef6a83bad62ae03a3d9bc8ae693b34926e53cc8c4de07df20ab22d'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')
# sha256sums_aarch64=()
validpgpkeys=('14F26682D0916CDD81E37B6D61B7B526D98F0353') # Mozilla Software Releases <release@mozilla.com>

# change this to false if you do not want to run a PGO build for aarch64 as well
_build_profiled_aarch64=true

# Fix some potential Python and a Rust error
if [ "${CC}" != "gcc" ] || [ "${CXX}" != "g++" ]; then
  export CC=gcc
  export CXX=g++
  export LD=ld
  export AS=""
  export NM=""
  export AR=""
  export RANLIB=""
  export OBJCOPY=""
  export LDFLAGS="${LDFLAGS/-static/}"
fi

prepare() {
  mkdir -p mozbuild
  cd firefox-"$pkgver"

  local _patches_dir
  _patches_dir="${srcdir}/common/patches"

  local _librewolf_patches_dir
  _librewolf_patches_dir="${srcdir}/librewolf-source/patches"

  local _cachyos_patches_dir
  _cachyos_patches_dir="${srcdir}/cachyos-source/patches"

  sed -i 's/lib\/librewolf/lib\/firedragon/g' "${_librewolf_patches_dir}/mozilla_dirs.patch"
  sed -i 's/lib64\/librewolf/lib64\/firedragon/g' "${_librewolf_patches_dir}/mozilla_dirs.patch"
  sed -i 's/librewolf/firedragon/g' "${_librewolf_patches_dir}/mozilla_dirs.patch"

  # Prepare patches, then return to the source directory
  pushd "${_patches_dir}" && sh "${srcdir}/common/rebrand.sh"
  popd
  pushd "${_librewolf_patches_dir}" && sh "${srcdir}/common/rebrand.sh"
  popd
  pushd "${_cachyos_patches_dir}" && sh "${srcdir}/common/rebrand.sh"
  popd


  cat >../mozconfig <<END
ac_add_options --enable-application=browser
mk_add_options MOZ_OBJDIR=${PWD@Q}/obj

# This supposedly speeds up compilation (We test through dogfooding anyway)
ac_add_options --disable-debug
ac_add_options --disable-tests

# Disables crash reporting, telemetry and other data gathering tools
mk_add_options MOZ_CRASHREPORTER=0
mk_add_options MOZ_DATA_REPORTING=0
mk_add_options MOZ_SERVICES_HEALTHREPORT=0
mk_add_options MOZ_TELEMETRY_REPORTING=0

ac_add_options --disable-bootstrap
ac_add_options --enable-default-toolkit=cairo-gtk3-x11-wayland
ac_add_options --enable-hardening
ac_add_options --enable-linker=mold
ac_add_options --enable-release
ac_add_options --enable-rust-simd
ac_add_options --prefix=/usr
ac_add_options --with-wasi-sysroot=/usr/share/wasi-sysroot

export AR=llvm-ar
export CC=clang
export CXX=clang++
export NM=llvm-nm
export RANLIB=llvm-ranlib

# Branding
ac_add_options --allow-addon-sideload
ac_add_options --enable-update-channel=release
ac_add_options --with-app-basename=${_pkgname}
ac_add_options --with-app-name=${__pkgname}
ac_add_options --with-branding=browser/branding/${__pkgname}
ac_add_options --with-distribution-id=org.garudalinux
ac_add_options --with-unsigned-addon-scopes=app,system
export MOZ_APP_REMOTINGNAME=${__pkgname//-/}

# System libraries
ac_add_options --with-system-icu
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# Features
ac_add_options --disable-crashreporter
ac_add_options --disable-debug
ac_add_options --disable-debug-js-modules
ac_add_options --disable-debug-symbols
ac_add_options --disable-gpsd
ac_add_options --disable-necko-wifi
ac_add_options --disable-rust-tests
ac_add_options --disable-synth-speechd
ac_add_options --disable-tests
ac_add_options --disable-trace-logging
ac_add_options --disable-updater
ac_add_options --disable-warnings-as-errors
ac_add_options --disable-webspeech
ac_add_options --disable-webspeechtestbackend
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-optimize=-O3
ac_add_options --enable-pulseaudio
ac_add_options --enable-strip
END

if [[ $CARCH == 'aarch64' ]]; then
  cat >>../mozconfig <<END
# taken from Manjaro build:
ac_add_options --enable-optimize="-g0 -O2"
END

  export MOZ_DEBUG_FLAGS=" "
  export CFLAGS+=" -g0"
  export CXXFLAGS+=" -g0"
  export RUSTFLAGS="-Cdebuginfo=0"
  export LDFLAGS+=" -Wl,--no-keep-memory"

else

  cat >>../mozconfig <<END
# probably not needed, enabled by default?
ac_add_options --enable-optimize

# Arch upstream has it in their PKGBUILD, ALARM does not for aarch64:
ac_add_options --disable-elf-hack

# Optimization
export CFLAGS="-march=znver2 -mtune=znver2 -O3 -fno-plt -fexceptions -ftree-vectorize -ftree-slp-vectorize -Wp,-D_FORTIFY_SOURCE=2 -Wformat -Wno-error -pthread -fstack-clash-protection -fcf-protection -fPIC -ffunction-sections -fdata-sections -fno-math-errno -pthread -pipe"
export LDFLAGS="-Wl,-O3,--sort-common,--as-needed,-z,relro,-z,now,-lgomp,-lpthread,--emit-relocs"
export RUSTFLAGS="-C opt-level=3 -C target-cpu=znver2"
END
fi

  # Upstream patches from gentoo (thanks CachyOS friends!)
  msg2 "---- Gentoo patches"
  for gentoo_patch in "${_cachyos_patches_dir}/gentoo/"*; do
    local _patch_name="`basename "${gentoo_patch}"`"
  if [ ${_patch_name} != "0031-bmo-1773259-cbindgen-root_clip_chain-fix.patch" ]; then
    msg2 "Patching with ${_patch_name}.."
    patch -Np1 -i "${gentoo_patch}"
    fi
  done

  # LibreWolf
  msg2 "---- Librewolf patches"
  # Remove some pre-installed addons that might be questionable
  patch -Np1 -i "${_librewolf_patches_dir}"/remove_addons.patch

  # Debian patch to enable global menubar
  patch -Np1 -i "${_librewolf_patches_dir}"/unity-menubar.patch

  # KDE menu
  patch -Np1 -i "${_librewolf_patches_dir}"/mozilla-kde_after_unity.patch
  patch -Np1 -i "${_cachyos_patches_dir}"/kde/mozilla-nongnome-proxies.patch

  # Disabling Pocket
  patch -Np1 -i "${_librewolf_patches_dir}"/sed-patches/disable-pocket.patch

  # Allow SearchEngines option in non-ESR builds
  patch -Np1 -i "${_librewolf_patches_dir}"/sed-patches/allow-searchengines-non-esr.patch

  # Remove search extensions (experimental)
  cp "${srcdir}/librewolf-source/assets/search-config.json" services/settings/dumps/main/search-config.json

  # Stop some undesired requests (https://gitlab.com/librewolf-community/browser/common/-/issues/10)
  patch -Np1 -i "${_librewolf_patches_dir}"/sed-patches/stop-undesired-requests.patch

  # Assorted patches
  patch -Np1 -i "${_librewolf_patches_dir}"/urlbarprovider-interventions.patch

  # Change some hardcoded directory strings that could lead to unnecessarily
  # created directories
  patch -Np1 -i "${_librewolf_patches_dir}"/mozilla_dirs.patch

  # Somewhat experimental patch to fix bus/dbus/remoting names to io.gitlab.librewolf
  # should not break things, buuuuuuuuuut we'll see.
  # patch -Np1 -i "${_librewolf_patches_dir}"/dbus_name.patch

  # Allow uBlockOrigin to run in private mode by default, without user intervention.
  patch -Np1 -i "${_librewolf_patches_dir}"/allow-ubo-private-mode.patch

  # Add custom uBO assets (on first launch only)
  patch -Np1 -i "${_librewolf_patches_dir}"/custom-ubo-assets-bootstrap-location.patch

  # Remove references to firefox from the settings UI, change text in some of the links,
  # explain that we force en-US and suggest enabling history near the session restore checkbox.
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/pref-naming.patch

  # Remap help links
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/remap-links.patch

  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/hide-default-browser.patch

  # Add LibreWolf logo to Debugging Page
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/lw-logo-devtools.patch

  # Update privecy preferences
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/privacy-preferences.patch

  # Remove firefox references in the urlbar, when suggesting opened tabs.
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/remove-branding-urlbar.patch

  # Remove cfr UI elements, as they are disabled and locked already.
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/remove-cfrprefs.patch

  # Do not display your browser is being managed by your organization in the settings.
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/remove-organization-policy-banner.patch

  # Hide "snippets" section from the home page settings, as it was already locked.
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/remove-snippets-from-home.patch

  # Add warning that sanitizing exceptions are bypassed by the options in History > Clear History when LibreWolf closes > Settings
  # patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/sanitizing-description.patch

  # Add patch to hide website appearance settings
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/website-appearance-ui-rfp.patch

  # Native massaging path
  # patch -Np1 -i "${_librewolf_patches_dir}"/native-messaging-registry-path.patch

  # Remove unncecessary handlers
  patch -Np1 -i "${_librewolf_patches_dir}"/ui-patches/handlers.patch

  # Fix telemetry removal, see https://gitlab.com/librewolf-community/browser/linux/-/merge_requests/17, for example
  patch -Np1 -i "${_librewolf_patches_dir}"/disable-data-reporting-at-compile-time.patch

  # Allows hiding the password manager (from the lw pref pane) / via a pref
  patch -Np1 -i "${_librewolf_patches_dir}"/hide-passwordmgr.patch

  # Faster multilocate
  patch -Np1 -i "${_librewolf_patches_dir}"/faster-package-multi-locale.patch

  # Pref pane - custom FireDragon svg
  patch -Np1 -i "${_patches_dir}"/custom/librewolf-pref-pane.patch
  patch -Np1 -i "${_patches_dir}"/custom/add_firedragon_svg.patch

  # Needed build fix
  # patch -Np1 -i "${_patches_dir}"/gentoo/0032-bmo-1773259-cbindgen-root_clip_chain-fix.patch

  # Mold linker patch by the CachyOS guys
  patch -Np1 -i "${_cachyos_patches_dir}"/add-mold-linker.patch
  patch -Np1 -i "${_cachyos_patches_dir}"/zstandard-0.18.0.patch
  patch -Np1 -i "${_patches_dir}"/custom/glibc236.patch
  patch -Np1 -i "${_patches_dir}"/custom/rust163.patch

  rm -f "${srcdir}"/common/source_files/mozconfig
  cp -r "${srcdir}"/common/source_files/* ./
}


build() {
  cd firefox-"$pkgver"

  export MOZ_NOSPAM=1
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  export MACH_BUILD_PYTHON_NATIVE_PACKAGE_SOURCE=pip
  export PIP_NETWORK_INSTALL_RESTRICTED_VIRTUALENVS=mach

  # LTO needs more open files
  ulimit -n 4096

  # Do 3-tier PGO
  echo "Building instrumented browser..."

  if [[ $CARCH == 'aarch64' ]]; then

    cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate
END

    else

    cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate=cross
END

  fi

  ./mach build

  echo "Profiling instrumented browser..."

  ./mach package

  LLVM_PROFDATA=llvm-profdata \
    JARLOG_FILE="$PWD/jarlog" \
    xvfb-run -s "-screen 0 1920x1080x24 -nolisten local" \
    ./mach python build/pgo/profileserver.py

  stat -c "Profile data found (%s bytes)" merged.profdata
  test -s merged.profdata

  stat -c "Jar log found (%s bytes)" jarlog
  test -s jarlog

  echo "Removing instrumented browser..."
  ./mach clobber

  echo "Building optimized browser..."

  if [[ $CARCH == 'aarch64' ]]; then

    cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-lto
ac_add_options --enable-profile-use
ac_add_options --with-pgo-profile-path=${PWD@Q}/merged.profdata
ac_add_options --with-pgo-jarlog=${PWD@Q}/jarlog
ac_add_options --enable-linker=lld
ac_add_options --disable-bootstrap
END

  else

    cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-lto=cross
ac_add_options --enable-profile-use=cross
ac_add_options --with-pgo-profile-path=${PWD@Q}/merged.profdata
ac_add_options --with-pgo-jarlog=${PWD@Q}/jarlog
ac_add_options --enable-linker=lld
ac_add_options --disable-bootstrap
END

  fi

  ./mach build

  echo "Building symbol archive..."
  ./mach buildsymbols
}

package() {
  cd firefox-"$pkgver"
  DESTDIR="$pkgdir" ./mach install

  rm "$pkgdir"/usr/lib/${__pkgname}/pingsender

  install -Dvm644 "$srcdir/settings/$__pkgname.psd" "$pkgdir/usr/share/psd/browsers/$__pkgname"

  local vendorjs
  vendorjs="$pkgdir/usr/lib/$__pkgname/browser/defaults/preferences/vendor.js"

  install -Dvm644 /dev/stdin "$vendorjs" <<END
// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Don't disable extensions in the application directory
// done in firedragon.cfg
// pref("extensions.autoDisableScopes", 11);
END

  # cd ${srcdir}/settings
  # git checkout ${_settings_commit}
  cd ${srcdir}/firefox-"$pkgver"
  cp -r ${srcdir}/settings/* ${pkgdir}/usr/lib/${__pkgname}/

  local distini="$pkgdir/usr/lib/$__pkgname/distribution/distribution.ini"
  install -Dvm644 /dev/stdin "$distini" <<END

[Global]
id=garudalinux
version=1.0
about=$_pkgname for Garuda Linux

[Preferences]
app.distributor=garudalinux
app.distributor.channel=$_pkgname
app.partner.garudalinux=garudalinux
END

  for i in 16 32 48 64 128; do
    install -Dvm644 browser/branding/${__pkgname}/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$__pkgname.png"
  done
  install -Dvm644 browser/branding/${__pkgname}/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$__pkgname.png"

  # Arch upstream provides a separate svg for this. we don't have that, so let's re-use 16.png
  install -Dvm644 browser/branding/${__pkgname}/default16.png \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$__pkgname-symbolic.png"

  install -Dvm644 ../$__pkgname.desktop \
    "$pkgdir/usr/share/applications/$__pkgname.desktop"

  # Install a wrapper to avoid confusion about binary path
  install -Dvm755 /dev/stdin "$pkgdir/usr/bin/$__pkgname" <<END
#!/bin/sh
exec /usr/lib/$__pkgname/$__pkgname "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srfv "$pkgdir/usr/bin/$__pkgname" "$pkgdir/usr/lib/$__pkgname/$__pkgname-bin"

  # Use system certificates
  local nssckbi="$pkgdir/usr/lib/$__pkgname/libnssckbi.so"
  if [[ -e $nssckbi ]]; then
    ln -srfv "$pkgdir/usr/lib/libnssckbi.so" "$nssckbi"
  fi

  # Make native messaging work
  ln -s "$pkgdir/usr/lib/mozilla/native-messaging-hosts" "$pkgdir/usr/lib/firedragon/native-messaging-hosts"

  # Delete unneeded things from settings repo
  rm "$pkgdir/usr/lib/firedragon/LICENSE.txt"
  rm "$pkgdir/usr/lib/firedragon/about.png"
  rm "$pkgdir/usr/lib/firedragon/firedragon.psd"
  rm "$pkgdir/usr/lib/firedragon/home.png"
  rm "$pkgdir/usr/lib/firedragon/package.json"
  rm "$pkgdir/usr/lib/firedragon/tabliss.json"
  rm "$pkgdir/usr/lib/firedragon/yarn.lock"
}
