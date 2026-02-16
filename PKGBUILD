# $Id$
# Maintainer: Philipp Marmet
# Contributor: Joe Julian

pkgname=mgmt
pkgver=1.0.1
pkgrel=3
pkgdesc='Next generation config management.'
arch=('x86_64' 'i686' 'armv6h' 'armv7h')
url="https://github.com/purpleidea/mgmt"
license=('GPL3')
makedepends=('augeas' 'go' 'go-md2man' 'go-tools' 'libpcap' 'libvirt' 'rubygems' 'ragel')
depends=('augeas' 'libvirt')
conflicts=('mgmt-bin')
backup=("etc/${pkgname}/${pkgname}.conf")
source=("${pkgname}-${pkgver}.tar.gz"::"${url}/archive/refs/tags/${pkgver}.tar.gz"
        "mgmt.openrc")
sha256sums=('1b0c8b6efc2c3064955d8dd3cadc6b11b9195ec07e33f0bf5e115a0e113494d6'
            'bc26455ddbf5d21be7598170011d69b4c1cd4529dd4b5ed1ca107f97941d261d')

prepare() {
  cd "${pkgname}-${pkgver}"
  # Redirect caches to srcdir so they are cleaned up automatically,
  # but don't set GOPATH which affects how paths are stored in binaries.
  export GOMODCACHE="${srcdir}/modcache"
  export GOCACHE="${srcdir}/go-build"

  # Download dependencies to the local cache
  go mod download -modcacherw
}

build() {
  export GOMODCACHE="${srcdir}/modcache"
  export GOCACHE="${srcdir}/go-build"

  # Arch-idiomatic flags that are safe for BOTH Linux and WASM:
  # -trimpath: Fixes the $srcdir warning
  # -buildvcs=false: Fixes the exit status 128 CI error
  export GOFLAGS="-trimpath -buildvcs=false -mod=readonly"

  cd "${srcdir}/${pkgname}-${pkgver}"

  # Use the Makefile for everything.
  # We pass MGMT_NOGOLANGRACE=true to keep the binary clean.
  # We do NOT add -buildmode=pie here to ensure the WASM UI builds correctly.
  make VERSION="${pkgver}" \
       SVERSION="${pkgver}" \
       PROGRAM="${pkgname}" \
       MGMT_NOGOLANGRACE=true

  # Clean up the module cache so it's not included in the package
  rm -rf "${GOMODCACHE}"
}

package() {
  install -d "${pkgdir}/run/${pkgname}"
  install -Dm755 "mgmt.openrc" "${pkgdir}/etc/init.d/mgmt"
  cd "${srcdir}/${pkgname}-${pkgver}"
  install -Dm755 "${pkgname}" "${pkgdir}/usr/bin/${pkgname}"
  install -Dm644 "misc/bashrc.sh" "${pkgdir}/usr/share/bash-completion/completions/${pkgname}"
  install -Dm644 "misc/example.conf" "${pkgdir}/etc/${pkgname}/${pkgname}.conf"
}
