# $Id$
# Maintainer: Timothy Redaelli <timothy.redaelli@gmail.com>
# Contributor: Zygmunt Krynicki <me at zygoon dot pl>

pkgbase=snapd
pkgname=snapd
pkgver=2.27.6
pkgrel=1
arch=('i686' 'x86_64')
url="https://github.com/snapcore/snapd"
license=('GPL3')
makedepends=('git' 'go' 'go-tools' 'libseccomp' 'libcap' 'python-docutils' 'systemd' 'xfsprogs' 'libseccomp')
checkdepends=('python' 'squashfs-tools' 'indent' 'shellcheck')

options=('!strip' 'emptydirs')
install=snapd.install
source=("https://github.com/snapcore/snapd/releases/download/$pkgver/${pkgname}_${pkgver}.vendor.tar.xz"
        'snapd.sh'
        '0001-cmd-snap-seccomp-link-to-libseccomp-dynamically.patch'
        '0002-snap-snapenv-backport-fixes-for-how-SNAP-is-computed.patch'
        '0003-release-remove-default-from-VERSION_ID.patch'
        '0004-snap-squashfs-use-LANG-C.UTF-8.patch'
        '0005-lirios-support.patch'
	'0006-streamcommand-tests.patch')

md5sums=('e701b2ae8d9e2155587939b34fbe8966'
         '8e9b8108165d5b2ae911de9caefb37ce'
         'eb6e9700e65a11f5aec4a2bf8aeebcd3'
         '53942daab6710e53d62a532d06ec9d10'
         'f6b013b7da25ed7ee1c0b90d66b90ab4'
         '066c7feb9e66dbed4d570bd146f28a5d'
         '02c8afa52621227364083a8967651be1'
         '1a5da6c08de75c3f251e768a911731e1')

_gourl=github.com/snapcore/snapd

prepare() {
  cd $pkgname-$pkgver

  # Use $srcdir/go as our GOPATH
  export GOPATH="$srcdir/go"
  mkdir -p "$GOPATH"
  # Have snapd checkout appear in a place suitable for subsequent GOPATH This
  # way we don't have to go get it again and it is exactly what the tag/hash
  # above describes.
  mkdir -p "$(dirname "$GOPATH/src/${_gourl}")"
  ln --no-target-directory -fs "$srcdir/$pkgname-$pkgver" "$GOPATH/src/${_gourl}"
  # Apply some patches that didn't make it into the release
  for p in $srcdir/*.patch; do
      patch -d "$srcdir/$pkgname-$pkgver" -p1 < $p
  done
}

build() {
  export GOPATH="$srcdir/go"
  cd "$GOPATH/src/${_gourl}"
  # Generate version
  ./mkversion.sh $pkgver-$pkgrel

  # Generate the real systemd units out of the available templates
  make -C data/systemd all

  # Build/install snap and snapd
  go install "${_gourl}/cmd/snap"
  go install "${_gourl}/cmd/snapctl"
  go install "${_gourl}/cmd/snapd"
  go install "${_gourl}/cmd/snap-update-ns"
  go install "${_gourl}/cmd/snap-seccomp"

  # Build snap-confine
  cd cmd
  autoreconf -i -f
  ./configure \
    --prefix=/usr \
    --libexecdir=/usr/lib/snapd \
    --disable-apparmor \
    --enable-nvidia-arch \
    --enable-merged-usr
  make
}

check() {
  export GOPATH="$srcdir/go"
  cd "$GOPATH/src/${_gourl}"
  go test ./...
  make -C cmd -k check
}

package() {
  pkgdesc="Service and tools for management of snap packages."
  depends=('snap-confine' 'squashfs-tools' 'libseccomp' 'libsystemd')
  replaces=('snap-confine')
  provides=('snap-confine')

  export GOPATH="$srcdir/go"
  # Ensure that we have /var/lib/snapd/{hostfs,lib/gl}/ as they are required by snap-confine
  # for constructing some bind mounts around.
  install -d -m 755 "$pkgdir/var/lib/snapd/hostfs/" "$pkgdir/var/lib/snapd/lib/gl/"
  # Install the refresh timer and service for updating snaps
  install -d -m 755 "$pkgdir/usr/lib/systemd/system/"
  install -m 644 "$GOPATH/src/${_gourl}/data/systemd/snapd.refresh.service" "$pkgdir/usr/lib/systemd/system"
  install -m 644 "$GOPATH/src/${_gourl}/data/systemd/snapd.refresh.timer" "$pkgdir/usr/lib/systemd/system"
  # Install the snapd socket and service for the main daemon
  install -m 644 "$GOPATH/src/${_gourl}/data/systemd/snapd.service" "$pkgdir/usr/lib/systemd/system"
  install -m 644 "$GOPATH/src/${_gourl}/data/systemd/snapd.socket" "$pkgdir/usr/lib/systemd/system"
  # Install snap, snapctl, snap-update-ns, snap-seccomp and snapd executables
  install -d -m 755 "$pkgdir/usr/bin/"
  install -m 755 "$GOPATH/bin/snap" "$pkgdir/usr/bin/"
  install -m 755 "$GOPATH/bin/snapctl" "$pkgdir/usr/bin/"
  install -d -m 755 "$pkgdir/usr/lib/snapd"
  install -m 755 "$GOPATH/bin/snap-update-ns" "$pkgdir/usr/lib/snapd/"
  install -m 755 "$GOPATH/bin/snap-seccomp" "$pkgdir/usr/lib/snapd/"
  install -m 755 "$GOPATH/bin/snapd" "$pkgdir/usr/lib/snapd/"
  # Install snap-confine
  make -C "$srcdir/$pkgname-$pkgver/cmd" install DESTDIR="$pkgdir/"
  # Install script to export binaries paths of snaps
  install -Dm 755 "$srcdir/snapd.sh" "$pkgdir/etc/profile.d/apps-bin-path.sh"
  # Install the bash tab completion files
  install -Dm 755 "$GOPATH/src/${_gourl}/data/completion/snap" "$pkgdir/usr/share/bash-completion/completions/snap"
  install -Dm 755 "$GOPATH/src/${_gourl}/data/completion/complete.sh" "$pkgdir/usr/lib/snapd/complete.sh"
  install -Dm 755 "$GOPATH/src/${_gourl}/data/completion/etelpmoc.sh" "$pkgdir/usr/lib/snapd/etelpmoc.sh"
}
