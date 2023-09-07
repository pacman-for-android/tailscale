# Maintainer: Morten Linderud <foxboron@archlinux.org>
# Maintainer: Christian Heusel <gromit@archlinux.org>
# Contributor: David Anderson <dave@natulte.net>

pkgname=tailscale
pkgver=1.48.1
pkgrel=2.5
pkgdesc="A mesh VPN that makes it easy to connect your devices, wherever they are."
arch=("x86_64" "aarch64")
url="https://tailscale.com"
license=("MIT")
makedepends=("git" "go")
depends=("glibc")
backup=("data/etc/default/tailscaled")
# Important: Check if the version has been published before updating
# curl -s "https://pkgs.tailscale.com/stable/?mode=json"
_commit=528f95da6a5e12961b36a39bb3fa512c093d2330 #  git rev-parse tags/v1.48.1
source=("git+https://github.com/tailscale/tailscale.git#commit=${_commit}"
        tailscale-change-prefix.patch)
sha256sums=('SKIP'
            '5dd06233acdd341d79e995c606a3fc7dee4e0394c8f1d2d9f8485370d9da88ee')

pkgver() {
  cd "${pkgname}"
  git describe --tags | sed 's/^[vV]//;s/-/+/g'
}

prepare() {
    cd "${pkgname}"
    patch -Np2 -i ../tailscale-change-prefix.patch
    go mod vendor
}

build() {
    cd "${pkgname}"
    export CGO_CPPFLAGS="${CPPFLAGS}"
    export CGO_CFLAGS="${CFLAGS}"
    export CGO_CXXFLAGS="${CXXFLAGS}"
    export CGO_LDFLAGS="${LDFLAGS}"
    # pacman bug
    # export GOPATH="${srcdir}"
    export GOFLAGS="-buildmode=pie -mod=readonly -modcacherw"
    GO_LDFLAGS="\
        -compressdwarf=false \
        -linkmode=external \
        -X tailscale.com/version.longStamp=${pkgver} \
        -X tailscale.com/version.shortStamp=$(cut -d+ -f1 <<< "${pkgver}") \
        -X tailscale.com/version.gitCommitStamp=${_commit}"
    for cmd in ./cmd/tailscale ./cmd/tailscaled; do
        go build -v -tags xversion -ldflags "$GO_LDFLAGS" "$cmd"
    done
}

#TODO: Figure out why tests are failing
# check() {
#     cd "${pkgname}"
#     go test $(go list ./... | grep -v tsdns_test)
# }

package() {
    cd "${pkgname}"
    install -Dm755 tailscale tailscaled -t "$pkgdir/data/usr/bin"
    install -Dm644 cmd/tailscaled/tailscaled.defaults "$pkgdir/data/etc/default/tailscaled"
    install -Dm644 cmd/tailscaled/tailscaled.service -t "$pkgdir/data/usr/lib/systemd/system"
    install -Dm644 LICENSE -t "$pkgdir/data/usr/share/licenses/$pkgname"
}
