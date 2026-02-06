# Maintainer: bkmo <>
pkgname=sdboot-snaps
pkgver=1.1
pkgrel=1
pkgdesc='Create UKIs for SD-Boot BTRFS bootable snapshot entries'
arch=('any')
license=('GPL3')
url='https://github.com/bkmo/sdboot-snaps'
depends=('systemd-ukify' 'snapper' 'btrfs-progs' 'coreutils')
optdepends=('sbctl')
provides=('sdboot-snaps')
source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/$pkgver.tar.gz")
sha256sums=('SKIP')
backup=("etc/sdboot-snaps.conf")

    package() {
    cd $pkgname-$pkgver
    install -Dm 0644  "pacman/05-path-stop.hook" -t "$pkgdir/usr/share/libalpm/hooks/"
    install -Dm 0644  "pacman/zz-path-start.hook" -t "$pkgdir/usr/share/libalpm/hooks/"
    install -Dm 0755  "pacman/snap-path-pre" -t "$pkgdir/usr/share/libalpm/scripts/"
    install -Dm 0755  "pacman/snap-path-post" -t "$pkgdir/usr/share/libalpm/scripts/"
    install -Dm 0644  "service/snapper-boot-entries.path" -t "$pkgdir/etc/systemd/system/"
    install -Dm 0644  "service/snapper-boot-entries.service" -t "$pkgdir/etc/systemd/system/"
    install -Dm 0755  "config/sdboot-snaps.conf" -t "$pkgdir/etc/"
    install -Dm 0755  "scripts/refresh-snapshot-ukis" -t "$pkgdir/usr/local/bin/"
    install -Dm 0755  "scripts/manage-snapshot-ukis" -t "$pkgdir/usr/local/bin/"
    install -Dm 0644   README.md "$pkgdir/usr/share/doc/sdboot-snaps/README.md"
    install -Dm755 detect/snapshot-detect -t "$pkgdir/usr/local/bin/"
    install -Dm644 detect/snapshot-detect.desktop -t "$pkgdir/etc/xdg/autostart/"

}
