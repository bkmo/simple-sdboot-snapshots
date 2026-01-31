# Maintainer: Gabby <28601 dash gabby at users dot noreply dot gitlab dot freedesktop dot org>
# Maintainer: Julien <aur dot arch at fastmail dot com>
# forked from https://github.com/jrabinow/snapper-rollback with bootbackup commits

pkgname=simple-sdboot-snapshots
pkg_name=sdboot-snaps
pkgver=1.0.1
pkgrel=1
pkgdesc='Create UKIs for SD-Boot BTRFS bootable snapshot entries'
arch=('any')
license=('GPL3')
url='https://github.com/bkmo/sdboot-snaps'
depends=('ukify' 'snapper' 'btrfs-progs')
optdepends=('sbctl')
provides=('simple-sdboot-snapshots')
source=(git+"$url")
sha256sums=('SKIP')

    package() {
    cd $pkg_name
    install -Dm 0644  "05-path-stop.hook" -t "$pkgdir/usr/share/libalpm/hooks/"
    install -Dm 0644  "zz-path-start.hook" -t "$pkgdir/usr/share/libalpm/hooks/"
    install -Dm 0755  "snap-path-pre" -t "$pkgdir/usr/share/libalpm/scripts/"
    install -Dm 0755  "snap-path-post" -t "$pkgdir/usr/share/libalpm/scripts/"
    install -Dm 0644  "snapper-boot-entries.path" -t "$pkgdir/etc/systemd/system/"
    install -Dm 0644  "snapper-boot-entries.service" -t "$pkgdir/etc/systemd/system/"
    install -Dm 0755  "sdboot-snaps.conf" -t "$pkgdir/etc/"
    install -Dm 0755  "refresh-snapshot-ukis" -t "$pkgdir/usr/local/bin/"
    install -Dm 0755  "manage-snapshot-ukis" -t "$pkgdir/usr/local/bin/"
}
