# $Id$
# Maintainer: Philipp Marmet
# Contributor: Joe Julian

pkgname=mgmt
pkgver=1.0.1
pkgrel=4
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
  # 1. Setup local caches to keep the build environment isolated
  export GOMODCACHE="${srcdir}/modcache"
  export GOCACHE="${srcdir}/go-build"

  # 2. Setup a local bin for the generators
  export GOBIN="${srcdir}/bin"
  export PATH="${GOBIN}:${PATH}"
  mkdir -p "${GOBIN}"

  # 3. Download app dependencies to the local cache
  cd "${pkgname}-${pkgver}"
  go mod download -modcacherw

  # 4. Install the generators into our local bin
  # We use -buildvcs=false to prevent the CI 'exit status 128' error here too
  go install github.com/blynn/nex@latest
  go install golang.org/x/tools/cmd/goyacc@latest      # formerly `go tool yacc`
  go install golang.org/x/tools/cmd/stringer@latest    # for automatic stringer-ing
  go install golang.org/x/lint/golint@latest           # for `golint`-ing
  go install golang.org/x/tools/cmd/goimports@latest   # for fmt
  go install github.com/dvyukov/go-fuzz/go-fuzz@latest # for fuzzing the mcl lang bits
}


build() {
  # 5. Setup local caches and paths
  export GOMODCACHE="${srcdir}/modcache"
  export GOCACHE="${srcdir}/go-build"
  export PATH="${srcdir}/bin:${PATH}"

  # 6. Standard Arch flags (safe for both Linux and WASM)
  # -trimpath: removes the $srcdir references
  # -buildvcs=false: we don't need the binary stamped with vcs revision
  export GOFLAGS="-trimpath -buildvcs=false -mod=readonly"

  cd "${srcdir}/${pkgname}-${pkgver}"

  # 7. Use the Makefile
  # MGMT_NOGOLANGRACE=true: Removes the hidden absolute paths in race-detector symbols
  make VERSION="${pkgver}" \
       SVERSION="${pkgver}" \
       PROGRAM="${pkgname}" \
       MGMT_NOGOLANGRACE=true

  sudo rm -rf "${GOMODCACHE}" "${srcdir}/bin"
}

package() {
  install -d "${pkgdir}/run/${pkgname}"
  install -Dm755 "mgmt.openrc" "${pkgdir}/etc/init.d/mgmt"
  cd "${srcdir}/${pkgname}-${pkgver}"
  install -Dm755 "${pkgname}" "${pkgdir}/usr/bin/${pkgname}"
  install -Dm644 "misc/bashrc.sh" "${pkgdir}/usr/share/bash-completion/completions/${pkgname}"
  install -Dm644 "misc/example.conf" "${pkgdir}/etc/${pkgname}/${pkgname}.conf"
}
