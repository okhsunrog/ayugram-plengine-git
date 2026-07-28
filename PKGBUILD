# Maintainer: okhsunrog <me@okhsunrog.dev>

: "${_branch:=linux-plengine}"
: "${_jobs:=10}"

pkgname=ayugram-plengine-git
pkgver=6.7.5.PLEngine.0.1.1.r4.g4cbb582
pkgrel=1
pkgdesc="AyuGram Desktop with the cross-platform PLEngine plugin system"
arch=('x86_64' 'aarch64')
url="https://github.com/okhsunrog/AyuGramDesktop-PLEngine"
license=('GPL-3.0-or-later')
depends=(
  ada
  ffmpeg
  hunspell
  kcoreaddons
  libavif
  libdispatch
  libheif
  libjxl
  libpipewire
  libvpx
  libxcomposite
  libxdamage
  libxrandr
  libxtst
  minizip-ng
  openal
  openh264
  opus
  protobuf
  qt6-base
  qt6-imageformats
  qt6-svg
  qt6-wayland
  rnnoise
  xcb-util-keysyms
  xxhash
)
makedepends=(
  boost
  boost-libs
  cmake
  extra-cmake-modules
  fmt
  git
  glib2-devel
  gobject-introspection
  gperf
  jemalloc
  libtg_owt
  ninja
  range-v3
  tl-expected
)
optdepends=(
  'webkit2gtk: embedded browser features'
  'xdg-desktop-portal: desktop integration'
)
provides=('ayugram-desktop')
conflicts=('ayugram-desktop')
options=('!lto')

_pkgsrc=AyuGramDesktop-PLEngine
_tdsrc=telegram-tdlib
source=(
  "$_pkgsrc::git+$url.git#branch=$_branch"
  "$_tdsrc::git+https://github.com/tdlib/td.git"
)
sha256sums=('SKIP' 'SKIP')

prepare() {
  cd "$_pkgsrc"

  git rm -r --ignore-unmatch \
    Telegram/ThirdParty/dispatch \
    Telegram/ThirdParty/hunspell \
    Telegram/ThirdParty/kcoreaddons \
    Telegram/ThirdParty/lz4 \
    Telegram/ThirdParty/range-v3
  git submodule update --init --recursive --depth=1

  rm -rf Telegram/ThirdParty/minizip
  sed -E -i \
    '/pkg_check_modules/ { /minizip-ng/! s/\bminizip\b/minizip-ng/; }' \
    cmake/external/minizip/CMakeLists.txt

  local file
  for file in \
    Telegram/ThirdParty/tgcalls/tgcalls/DirectConnectionChannel.h \
    Telegram/ThirdParty/tgcalls/tgcalls/third-party/json11.cpp \
    Telegram/ThirdParty/tgcalls/tgcalls/v2/SignalingConnection.h
  do
    grep -qxF '#include <cstdint>' "$file" \
      || sed -i '1i#include <cstdint>' "$file"
  done
}

pkgver() {
  cd "$_pkgsrc"

  git describe --long --tags --abbrev=7 --match='*-PLEngine-*' \
    | sed -E \
      's/^[^0-9]+//; s/-([0-9]+)-g/.r\1.g/; s/-/./g'
}

build() {
  local tde2e_options=(
    -B build_tde2e
    -S "$_tdsrc"
    -G Ninja
    -DCMAKE_BUILD_TYPE=None
    -DCMAKE_INSTALL_PREFIX=/usr
    -DTD_E2E_ONLY=ON
    -DBUILD_SHARED_LIBS=OFF
    -DBUILD_TESTING=OFF
    -Wno-dev
  )
  cmake "${tde2e_options[@]}"
  cmake --build build_tde2e --parallel "$_jobs"
  DESTDIR="$srcdir/deps" cmake --install build_tde2e

  local ayugram_options=(
    -B build
    -S "$_pkgsrc"
    -G Ninja
    -DCMAKE_BUILD_TYPE=None
    -DCMAKE_INSTALL_PREFIX=/usr
    -DCMAKE_PREFIX_PATH="$srcdir/deps/usr"
    -DDESKTOP_APP_DISABLE_AUTOUPDATE=ON
    -DTDESKTOP_API_ID=611335
    -DTDESKTOP_API_HASH=d524b414d21f4d37f08684c1df41ac9c
    -DDESKTOP_APP_USE_PACKAGED_FONTS=OFF
    -Wno-dev
  )
  cmake "${ayugram_options[@]}"
  cmake --build build --parallel "$_jobs"
}

package() {
  DESTDIR="$pkgdir" cmake --install build
}
